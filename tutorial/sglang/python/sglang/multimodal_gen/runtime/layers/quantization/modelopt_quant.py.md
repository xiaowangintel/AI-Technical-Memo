# modelopt_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/modelopt_quant.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `ModelOptQuantConfig`, `ModelOptFp8Config`, and `ModelOptFp4Config`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `ModelOptQuantConfig`、`ModelOptFp8Config` 和 `ModelOptFp4Config` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-40: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

import logging
import re
from functools import lru_cache
from typing import Any, Dict, List, Optional

import torch

from sglang.multimodal_gen.runtime.layers.linear import (
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.multimodal_gen.runtime.models.parameter import (
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.multimodal_gen.runtime.models.utils import set_weight_attrs
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.srt.layers.quantization.fp8_utils import (
    apply_fp8_linear,
    cutlass_fp8_supported,
)
from sglang.srt.layers.quantization.modelopt_quant import (
    pad_nvfp4_activation_for_cutlass,
    pad_nvfp4_weight,
    slice_nvfp4_output,
)
from sglang.srt.layers.quantization.utils import (
    convert_to_channelwise,
    is_layer_skipped,
    requantize_with_max_scale,
)
from sglang.srt.layers.utils.common import copy_or_rebind_param
from sglang.srt.utils.common import is_flashinfer_available, round_up
```
**EN:** This block establishes the module context and imports `__future__`, `logging`, `re`, `functools`, `typing`, and `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`logging`、`re`、`functools`、`typing` 和 `torch`。这些依赖为后续实现提供所需符号。

### Lines 42-47: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)

if is_flashinfer_available():
    import flashinfer
else:
    flashinfer = None
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`, and `is_flashinfer_available`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 和 `is_flashinfer_available` 协同工作。

### Lines 50-52: `_get_fp4_quantize_op` implementation / `_get_fp4_quantize_op` 实现
```python
@lru_cache(maxsize=1)
def _get_fp4_quantize_op():
    return current_platform.get_modelopt_fp4_quantize_op()
```
**EN:** This block defines function `_get_fp4_quantize_op`. It retrieves fp4 quantize op. Key calls include `lru_cache`, and `current_platform.get_modelopt_fp4_quantize_op`.
**CN:** 该代码块定义了函数 `_get_fp4_quantize_op`。 它用于获取fp4 quantize op。 关键调用包括 `lru_cache` 和 `current_platform.get_modelopt_fp4_quantize_op`。

### Lines 55-57: `_get_fp4_gemm_op` implementation / `_get_fp4_gemm_op` 实现
```python
@lru_cache(maxsize=1)
def _get_fp4_gemm_op():
    return current_platform.get_modelopt_fp4_gemm_op()
```
**EN:** This block defines function `_get_fp4_gemm_op`. It retrieves fp4 gemm op. Key calls include `lru_cache`, and `current_platform.get_modelopt_fp4_gemm_op`.
**CN:** 该代码块定义了函数 `_get_fp4_gemm_op`。 它用于获取fp4 gemm op。 关键调用包括 `lru_cache` 和 `current_platform.get_modelopt_fp4_gemm_op`。

### Lines 60-66: `_prepare_nvfp4_weight_bytes` implementation / `_prepare_nvfp4_weight_bytes` 实现
```python
def _prepare_nvfp4_weight_bytes(
    weight: torch.Tensor, *, swap_weight_nibbles: bool
) -> torch.Tensor:
    """Normalize serialized NVFP4 bytes before padding for the runtime kernel."""
    if not swap_weight_nibbles:
        return weight.contiguous()
    return ((weight >> 4) | (weight << 4)).contiguous()
```
**EN:** This block defines function `_prepare_nvfp4_weight_bytes`. Normalize serialized NVFP4 bytes before padding for the runtime kernel. Key calls include `contiguous`, and `weight.contiguous`. The implementation branches on conditions. Parameters such as `weight` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_prepare_nvfp4_weight_bytes`。 它用于准备nvfp4 weight bytes。 关键调用包括 `contiguous` 和 `weight.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `weight` 等参数驱动。

### Lines 69-74: `_require_flashinfer` implementation / `_require_flashinfer` 实现
```python
def _require_flashinfer():
    if flashinfer is None:
        raise RuntimeError(
            "flashinfer is required for the diffusion NVFP4 FlashInfer path."
        )
    return flashinfer
```
**EN:** This block defines function `_require_flashinfer`. It handles require flashinfer logic. Key calls include `RuntimeError`. The implementation branches on conditions.
**CN:** 该代码块定义了函数 `_require_flashinfer`。 它用于处理 require flashinfer 相关逻辑。 关键调用包括 `RuntimeError`。 实现中包含条件分支。

### Lines 77-77: `ModelOptQuantConfig` class overview / `ModelOptQuantConfig` 类概览
```python
class ModelOptQuantConfig(QuantizationConfig):
```
**EN:** This block defines class `ModelOptQuantConfig`. It encapsulates model opt quant config behavior. It inherits from `QuantizationConfig`.
**CN:** 该代码块定义了类 `ModelOptQuantConfig`。 它用于封装 model opt quant config 相关行为。 它继承自 `QuantizationConfig`。

### Lines 78-85: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        exclude_modules: Optional[List[str]],
        packed_modules_mapping: Optional[Dict[str, List[str]]],
    ):
        super().__init__()
        self.packed_modules_mapping = packed_modules_mapping or {}
        self.exclude_modules = exclude_modules or []
```
**EN:** This block defines method `__init__` on `ModelOptQuantConfig`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `exclude_modules`, and `packed_modules_mapping` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptQuantConfig` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `exclude_modules` 和 `packed_modules_mapping` 等参数驱动。

### Lines 87-103: `_get_quant_method` implementation / `_get_quant_method` 实现
```python
    def _get_quant_method(
        self,
        layer: torch.nn.Module,
        prefix: str,
        *,
        Linear: type[LinearMethodBase],
    ) -> Optional[QuantizeMethodBase]:
        from sglang.multimodal_gen.runtime.layers.linear import LinearBase

        if isinstance(layer, LinearBase):
            if self.is_layer_excluded(prefix) or (
                self.packed_modules_mapping
                and is_layer_skipped(prefix, [], self.packed_modules_mapping)
            ):
                return UnquantizedLinearMethod()
            return Linear(self)
        return None
```
**EN:** This block defines method `_get_quant_method` on `ModelOptQuantConfig`. It retrieves quant method. Key calls include `isinstance`, `Linear`, `self.is_layer_excluded`, `UnquantizedLinearMethod`, and `is_layer_skipped`. The implementation branches on conditions. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptQuantConfig` 的方法 `_get_quant_method`。 它用于获取quant method。 关键调用包括 `isinstance`、`Linear`、`self.is_layer_excluded`、`UnquantizedLinearMethod` 和 `is_layer_skipped`。 实现中包含条件分支。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 105-107: `get_config_filenames` implementation / `get_config_filenames` 实现
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return ["hf_quant_config.json"]
```
**EN:** This block defines method `get_config_filenames` on `ModelOptQuantConfig`. It retrieves config filenames.
**CN:** 该代码块定义了 `ModelOptQuantConfig` 的方法 `get_config_filenames`。 它用于获取config filenames。

### Lines 109-110: `get_scaled_act_names` implementation / `get_scaled_act_names` 实现
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines method `get_scaled_act_names` on `ModelOptQuantConfig`. It retrieves scaled act names.
**CN:** 该代码块定义了 `ModelOptQuantConfig` 的方法 `get_scaled_act_names`。 它用于获取scaled act names。

### Lines 112-128: `override_quantization_method` implementation / `override_quantization_method` 实现
```python
    @classmethod
    def override_quantization_method(cls, hf_quant_config, user_quant) -> Optional[str]:
        if hf_quant_config is None:
            return None

        quant_algo = (
            hf_quant_config.get("quant_algo")
            or hf_quant_config.get("quantization", {}).get("quant_algo")
            or ""
        ).upper()
        if user_quant in {"modelopt", "modelopt_fp8"} and "FP8" in quant_algo:
            return "modelopt_fp8"
        if user_quant in {"modelopt", "modelopt_fp4"} and (
            "NVFP4" in quant_algo or "FP4" in quant_algo
        ):
            return "modelopt_fp4"
        return None
```
**EN:** This block defines method `override_quantization_method` on `ModelOptQuantConfig`. It handles override quantization method logic. Key calls include `upper`, `hf_quant_config.get`, and `hf_quant_config.get.get`. The implementation branches on conditions. Parameters such as `hf_quant_config`, and `user_quant` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptQuantConfig` 的方法 `override_quantization_method`。 它用于处理 override quantization method 相关逻辑。 关键调用包括 `upper`、`hf_quant_config.get` 和 `hf_quant_config.get.get`。 实现中包含条件分支。 本段逻辑主要由 `hf_quant_config` 和 `user_quant` 等参数驱动。

### Lines 130-135: `is_layer_excluded` implementation / `is_layer_excluded` 实现
```python
    def is_layer_excluded(self, prefix: str) -> bool:
        for pattern in self.exclude_modules:
            regex_str = re.escape(pattern).replace(r"\*", r".*")
            if re.fullmatch(regex_str, prefix):
                return True
        return False
```
**EN:** This block defines method `is_layer_excluded` on `ModelOptQuantConfig`. It handles is layer excluded logic. Key calls include `re.escape.replace`, `re.fullmatch`, and `re.escape`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptQuantConfig` 的方法 `is_layer_excluded`。 它用于处理 is layer excluded 相关逻辑。 关键调用包括 `re.escape.replace`、`re.fullmatch` 和 `re.escape`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `prefix` 等参数驱动。

### Lines 138-140: `ModelOptFp8Config` class overview / `ModelOptFp8Config` 类概览
```python
class ModelOptFp8Config(ModelOptQuantConfig):
    """Config class for ModelOpt FP8 diffusion checkpoints."""
```
**EN:** This block defines class `ModelOptFp8Config`. Config class for ModelOpt FP8 diffusion checkpoints. It inherits from `ModelOptQuantConfig`.
**CN:** 该代码块定义了类 `ModelOptFp8Config`。 它用于封装 model opt fp8 config 相关行为。 它继承自 `ModelOptQuantConfig`。

### Lines 141-152: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        is_checkpoint_fp8_serialized: bool = False,
        exclude_modules: Optional[List[str]] = None,
        packed_modules_mapping: Optional[Dict[str, List[str]]] = None,
    ) -> None:
        super().__init__(exclude_modules, packed_modules_mapping)
        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized
        if is_checkpoint_fp8_serialized:
            logger.warning(
                "Detected ModelOpt FP8 checkpoint. The format is experimental and subject to change."
            )
```
**EN:** This block defines method `__init__` on `ModelOptFp8Config`. It initializes the instance state. Key calls include `super.__init__`, `logger.warning`, and `super`. The implementation branches on conditions. Parameters such as `is_checkpoint_fp8_serialized`, `exclude_modules`, and `packed_modules_mapping` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`logger.warning` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `is_checkpoint_fp8_serialized`、`exclude_modules` 和 `packed_modules_mapping` 等参数驱动。

### Lines 154-156: `get_name` implementation / `get_name` 实现
```python
    @classmethod
    def get_name(cls) -> str:
        return "modelopt_fp8"
```
**EN:** This block defines method `get_name` on `ModelOptFp8Config`. It retrieves name.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_name`。 它用于获取name。

### Lines 158-160: `get_supported_act_dtypes` implementation / `get_supported_act_dtypes` 实现
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.bfloat16, torch.half]
```
**EN:** This block defines method `get_supported_act_dtypes` on `ModelOptFp8Config`. It retrieves supported act dtypes.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_supported_act_dtypes`。 它用于获取supported act dtypes。

### Lines 162-164: `get_min_capability` implementation / `get_min_capability` 实现
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 89
```
**EN:** This block defines method `get_min_capability` on `ModelOptFp8Config`. It retrieves min capability.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_min_capability`。 它用于获取min capability。

### Lines 166-189: `from_config` implementation / `from_config` 实现
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> "ModelOptFp8Config":
        quant_method = config.get("quant_algo")
        exclude_modules = config.get("ignore")
        if quant_method is None:
            try:
                quantization_section = cls.get_from_keys(config, ["quantization"])
                quant_method = quantization_section.get("quant_algo")
                exclude_modules = quantization_section.get("exclude_modules")
            except ValueError as exc:
                raise ValueError(
                    "Cannot find 'quant_algo' in the model's quantization config."
                ) from exc

        if quant_method is None or "FP8" not in quant_method:
            raise ValueError(
                "ModelOptFp8Config only supports static FP8 quantization in SGLang diffusion."
            )

        return cls(
            is_checkpoint_fp8_serialized=True,
            exclude_modules=exclude_modules,
            packed_modules_mapping=config.get("packed_modules_mapping"),
        )
```
**EN:** This block defines method `from_config` on `ModelOptFp8Config`. It constructs from config. Key calls include `config.get`, `cls`, `ValueError`, `cls.get_from_keys`, and `quantization_section.get`. The implementation branches on conditions, handles exceptional paths. Parameters such as `config` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `from_config`。 它用于从…构造config。 关键调用包括 `config.get`、`cls`、`ValueError`、`cls.get_from_keys` 和 `quantization_section.get`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `config` 等参数驱动。

### Lines 191-192: `get_quant_method` implementation / `get_quant_method` 实现
```python
    def get_quant_method(self, layer: torch.nn.Module, prefix: str):
        return self._get_quant_method(layer, prefix, Linear=ModelOptFp8LinearMethod)
```
**EN:** This block defines method `get_quant_method` on `ModelOptFp8Config`. It retrieves quant method. Key calls include `self._get_quant_method`. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_quant_method`。 它用于获取quant method。 关键调用包括 `self._get_quant_method`。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 195-197: `ModelOptFp4Config` class overview / `ModelOptFp4Config` 类概览
```python
class ModelOptFp4Config(ModelOptQuantConfig):
    """Config class for NVFP4."""
```
**EN:** This block defines class `ModelOptFp4Config`. Config class for NVFP4. It inherits from `ModelOptQuantConfig`.
**CN:** 该代码块定义了类 `ModelOptFp4Config`。 它用于封装 model opt fp4 config 相关行为。 它继承自 `ModelOptQuantConfig`。

### Lines 198-216: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        is_checkpoint_nvfp4_serialized: bool = False,
        group_size: int = None,
        exclude_modules: List[str] = None,
        packed_modules_mapping: Optional[Dict[str, List[str]]] = None,
        checkpoint_uses_packed_qkv: bool = False,
        swap_weight_nibbles: bool = True,
    ) -> None:
        super().__init__(exclude_modules, packed_modules_mapping)
        self.is_checkpoint_nvfp4_serialized = is_checkpoint_nvfp4_serialized
        if is_checkpoint_nvfp4_serialized:
            logger.warning(
                "Detected nvfp4 checkpoint. Please note that the "
                "format is experimental and subject to change."
            )
        self.group_size = group_size
        self.checkpoint_uses_packed_qkv = checkpoint_uses_packed_qkv
        self.swap_weight_nibbles = swap_weight_nibbles
```
**EN:** This block defines method `__init__` on `ModelOptFp4Config`. It initializes the instance state. Key calls include `super.__init__`, `logger.warning`, and `super`. The implementation branches on conditions. Parameters such as `is_checkpoint_nvfp4_serialized`, `group_size`, `exclude_modules`, `packed_modules_mapping`, and `checkpoint_uses_packed_qkv` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp4Config` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`logger.warning` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `is_checkpoint_nvfp4_serialized`、`group_size`、`exclude_modules`、`packed_modules_mapping` 和 `checkpoint_uses_packed_qkv` 等参数驱动。

### Lines 218-220: `get_name` implementation / `get_name` 实现
```python
    @classmethod
    def get_name(cls) -> str:
        return "modelopt_fp4"
```
**EN:** This block defines method `get_name` on `ModelOptFp4Config`. It retrieves name.
**CN:** 该代码块定义了 `ModelOptFp4Config` 的方法 `get_name`。 它用于获取name。

### Lines 222-224: `get_supported_act_dtypes` implementation / `get_supported_act_dtypes` 实现
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.bfloat16, torch.half, torch.float8_e4m3fn]
```
**EN:** This block defines method `get_supported_act_dtypes` on `ModelOptFp4Config`. It retrieves supported act dtypes.
**CN:** 该代码块定义了 `ModelOptFp4Config` 的方法 `get_supported_act_dtypes`。 它用于获取supported act dtypes。

### Lines 226-228: `get_min_capability` implementation / `get_min_capability` 实现
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 100
```
**EN:** This block defines method `get_min_capability` on `ModelOptFp4Config`. It retrieves min capability.
**CN:** 该代码块定义了 `ModelOptFp4Config` 的方法 `get_min_capability`。 它用于获取min capability。

### Lines 230-258: `common_group_size` implementation / `common_group_size` 实现
```python
    @staticmethod
    def common_group_size(cfg: dict) -> int:
        """Return the unique group_size across the config; raise if missing/mismatched."""
        sizes = set()

        def _add_group_size_from_dict(config: dict):
            group_size = config.get("group_size")
            if isinstance(group_size, int):
                sizes.add(group_size)

        # Top-level and 'quantization' block
        _add_group_size_from_dict(cfg)
        quantization = cfg.get("quantization")
        if isinstance(quantization, dict):
            _add_group_size_from_dict(quantization)

        # config_groups: accept group-level or nested dicts (e.g., weights/input_activations)
        for config_groups in (cfg.get("config_groups") or {}).values():
            if isinstance(config_groups, dict):
                _add_group_size_from_dict(config_groups)
                for config_group in config_groups.values():
                    if isinstance(config_group, dict):
                        _add_group_size_from_dict(config_group)

        if not sizes:
            raise ValueError("No group_size found in config.")
        if len(sizes) > 1:
            raise ValueError(f"Inconsistent group_size values: {sorted(sizes)}")
        return next(iter(sizes))
```
**EN:** This block defines method `common_group_size` on `ModelOptFp4Config`. Return the unique group_size across the config; raise if missing/mismatched. Key calls include `set`, `_add_group_size_from_dict`, `cfg.get`, `isinstance`, and `values`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `cfg` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp4Config` 的方法 `common_group_size`。 它用于处理 common group size 相关逻辑。 关键调用包括 `set`、`_add_group_size_from_dict`、`cfg.get`、`isinstance` 和 `values`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `cfg` 等参数驱动。

### Lines 260-308: `from_config` implementation / `from_config` 实现
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> ModelOptFp4Config:
        group_size = None
        exclude_modules = []
        swap_weight_nibbles = True

        # Flat format (config.json quantization_config)
        quant_method = config.get("quant_algo")
        if quant_method is not None:
            group_size = config.get("group_size")
            if group_size is None:
                config_groups = config.get("config_groups", {})
                if config_groups:
                    first_group = next(iter(config_groups.values()), {})
                    group_size = first_group.get("weights", {}).get("group_size")
            exclude_modules = config.get("ignore", [])
            swap_weight_nibbles = config.get("swap_weight_nibbles", True)
        else:
            # Nested format (hf_quant_config.json)
            try:
                quant_config = cls.get_from_keys(config, ["quantization"])
                quant_method = quant_config["quant_algo"]
                group_size = ModelOptFp4Config.common_group_size(config)
                exclude_modules = quant_config.get("exclude_modules", [])
                swap_weight_nibbles = quant_config.get(
                    "swap_weight_nibbles",
                    config.get("swap_weight_nibbles", True),
                )
            except (ValueError, KeyError):
                raise ValueError("Cannot find 'quant_algo' in quantization config.")

        if quant_method not in ["NVFP4"]:
            raise ValueError(
                f"Only NVFP4 quantization is supported for diffusion, got '{quant_method}'."
            )

        if group_size is None or exclude_modules is None:
            raise ValueError(
                "NVFP4 quantization requires group_size and exclude_modules "
                "in the quantization config"
            )
        return cls(
            is_checkpoint_nvfp4_serialized=True,
            group_size=group_size,
            exclude_modules=exclude_modules,
            packed_modules_mapping=config.get("packed_modules_mapping"),
            checkpoint_uses_packed_qkv=config.get("checkpoint_uses_packed_qkv", False),
            swap_weight_nibbles=swap_weight_nibbles,
        )
```
**EN:** This block defines method `from_config` on `ModelOptFp4Config`. It constructs from config. Key calls include `config.get`, `cls`, `ValueError`, `cls.get_from_keys`, and `ModelOptFp4Config.common_group_size`. The implementation branches on conditions, handles exceptional paths. Parameters such as `config` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp4Config` 的方法 `from_config`。 它用于从…构造config。 关键调用包括 `config.get`、`cls`、`ValueError`、`cls.get_from_keys` 和 `ModelOptFp4Config.common_group_size`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `config` 等参数驱动。

### Lines 310-311: `get_quant_method` implementation / `get_quant_method` 实现
```python
    def get_quant_method(self, layer: torch.nn.Module, prefix: str):
        return self._get_quant_method(layer, prefix, Linear=ModelOptFp4LinearMethod)
```
**EN:** This block defines method `get_quant_method` on `ModelOptFp4Config`. It retrieves quant method. Key calls include `self._get_quant_method`. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp4Config` 的方法 `get_quant_method`。 它用于获取quant method。 关键调用包括 `self._get_quant_method`。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 314-316: `ModelOptFp8LinearMethod` class overview / `ModelOptFp8LinearMethod` 类概览
```python
class ModelOptFp8LinearMethod(LinearMethodBase):
    """Linear method for ModelOpt static FP8 checkpoints."""
```
**EN:** This block defines class `ModelOptFp8LinearMethod`. Linear method for ModelOpt static FP8 checkpoints. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `ModelOptFp8LinearMethod`。 它用于封装 model opt fp8 linear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 317-319: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, quant_config: ModelOptFp8Config):
        self.quant_config = quant_config
        self.cutlass_fp8_supported = cutlass_fp8_supported()
```
**EN:** This block defines method `__init__` on `ModelOptFp8LinearMethod`. It initializes the instance state. Key calls include `cutlass_fp8_supported`. Parameters such as `quant_config` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8LinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `cutlass_fp8_supported`。 本段逻辑主要由 `quant_config` 等参数驱动。

### Lines 321-370: `create_weights` implementation / `create_weights` 实现
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: List[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        del input_size, output_size
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        weight_dtype = (
            torch.float8_e4m3fn
            if self.quant_config.is_checkpoint_fp8_serialized
            else params_dtype
        )
        layer.register_parameter(
            "weight",
            ModelWeightParameter(
                data=torch.empty(
                    output_size_per_partition,
                    input_size_per_partition,
                    dtype=weight_dtype,
                ),
                input_dim=1,
                output_dim=0,
                weight_loader=weight_loader,
            ),
        )

        if self.quant_config.is_checkpoint_fp8_serialized:
            for scale_name in ["weight_scale", "input_scale"]:
                layer.register_parameter(
                    scale_name,
                    PerTensorScaleParameter(
                        data=torch.full(
                            (len(output_partition_sizes),),
                            torch.finfo(torch.float32).min,
                            dtype=torch.float32,
                        ),
                        weight_loader=weight_loader,
                    ),
                )
```
**EN:** This block defines method `create_weights` on `ModelOptFp8LinearMethod`. It creates weights. Key calls include `sum`, `extra_weight_attrs.get`, `layer.register_parameter`, `ModelWeightParameter`, and `torch.empty`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8LinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `sum`、`extra_weight_attrs.get`、`layer.register_parameter`、`ModelWeightParameter` 和 `torch.empty`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 372-383: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        max_w_scale, quantized_weight = requantize_with_max_scale(
            layer.weight, layer.weight_scale, layer.logical_widths
        )
        # Preserve the parameter subclass metadata while rebinding to the
        # transposed FP8 view expected by the runtime.
        layer.weight.data = quantized_weight.t().detach()
        layer.weight.requires_grad_(False)
        if self.cutlass_fp8_supported:
            max_w_scale = convert_to_channelwise(max_w_scale, layer.logical_widths)
        copy_or_rebind_param(layer, "weight_scale", max_w_scale)
        copy_or_rebind_param(layer, "input_scale", layer.input_scale.max())
```
**EN:** This block defines method `process_weights_after_loading` on `ModelOptFp8LinearMethod`. It processes weights after loading. Key calls include `requantize_with_max_scale`, `quantized_weight.t.detach`, `layer.weight.requires_grad_`, `copy_or_rebind_param`, and `convert_to_channelwise`. The implementation branches on conditions. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8LinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `requantize_with_max_scale`、`quantized_weight.t.detach`、`layer.weight.requires_grad_`、`copy_or_rebind_param` 和 `convert_to_channelwise`。 实现中包含条件分支。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 385-398: `apply` implementation / `apply` 实现
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return apply_fp8_linear(
            input=x,
            weight=layer.weight,
            weight_scale=layer.weight_scale,
            input_scale=layer.input_scale,
            bias=bias,
            cutlass_fp8_supported=self.cutlass_fp8_supported,
        )
```
**EN:** This block defines method `apply` on `ModelOptFp8LinearMethod`. It applies function. Key calls include `apply_fp8_linear`. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8LinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `apply_fp8_linear`。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

### Lines 401-403: `ModelOptFp4LinearMethod` class overview / `ModelOptFp4LinearMethod` 类概览
```python
class ModelOptFp4LinearMethod(LinearMethodBase):
    """NVFP4 linear method using CUTLASS FP4 GEMM."""
```
**EN:** This block defines class `ModelOptFp4LinearMethod`. NVFP4 linear method using CUTLASS FP4 GEMM. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `ModelOptFp4LinearMethod`。 它用于封装 model opt fp4 linear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 404-405: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, quant_config: ModelOptFp4Config):
        self.quant_config = quant_config
```
**EN:** This block defines method `__init__` on `ModelOptFp4LinearMethod`. It initializes the instance state. Parameters such as `quant_config` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp4LinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `quant_config` 等参数驱动。

### Lines 407-479: `create_weights` implementation / `create_weights` 实现
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: List[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        del input_size, output_size
        if not self.quant_config.is_checkpoint_nvfp4_serialized:
            raise ValueError(
                "NVFP4 quantization was selected, "
                " dynamic quantization is not supported."
            )
        if input_size_per_partition % 16 != 0:
            raise ValueError(
                f"Unsupported model when input features size is {input_size_per_partition}, not multiple of 16, for NVFP4 quantization."
            )

        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        layer.logical_widths = output_partition_sizes

        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        weight_dtype = (
            torch.float8_e4m3fn
            if self.quant_config.is_checkpoint_nvfp4_serialized
            else params_dtype
        )

        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // 2,
                dtype=torch.uint8,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        input_scale = PerTensorScaleParameter(
            data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
            weight_loader=weight_loader,
        )
        set_weight_attrs(input_scale, {"missing_param_init": "ones"})
        layer.register_parameter("input_scale", input_scale)

        weight_scale_2 = PerTensorScaleParameter(
            data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
            weight_loader=weight_loader,
        )
        set_weight_attrs(weight_scale_2, {"missing_param_init": "ones"})
        layer.register_parameter("weight_scale_2", weight_scale_2)

        weight_scale = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // self.quant_config.group_size,
                dtype=weight_dtype,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        set_weight_attrs(weight_scale, {"missing_param_init": "ones"})
        layer.register_parameter("weight_scale", weight_scale)
```
**EN:** This block defines method `create_weights` on `ModelOptFp4LinearMethod`. It creates weights. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, and `PerTensorScaleParameter`. The implementation branches on conditions. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp4LinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `sum`、`extra_weight_attrs.get`、`ModelWeightParameter`、`layer.register_parameter` 和 `PerTensorScaleParameter`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 481-570: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        input_scale_2 = layer.input_scale.max().to(torch.float32)
        weight_scale_2 = layer.weight_scale_2.max().to(torch.float32)

        copy_or_rebind_param(
            layer, "alpha", (input_scale_2 * weight_scale_2).to(torch.float32)
        )
        copy_or_rebind_param(
            layer, "input_scale_inv", (1 / input_scale_2).to(torch.float32)
        )

        layer.output_size_per_partition = layer.weight.shape[0]

        w = layer.weight.data
        w_swapped = _prepare_nvfp4_weight_bytes(
            w,
            swap_weight_nibbles=getattr(self.quant_config, "swap_weight_nibbles", True),
        )

        _, flashinfer_backend = _get_fp4_gemm_op()
        if flashinfer_backend == "trtllm":
            flashinfer_ops = _require_flashinfer()

            weight, _ = pad_nvfp4_weight(w_swapped, n_alignment=128, k_alignment=0)
            scales = layer.weight_scale
            if scales.shape[0] != weight.shape[0]:
                pad_n = weight.shape[0] - scales.shape[0]
                scales = torch.nn.functional.pad(scales, (0, 0, 0, pad_n))

            scale_k = scales.shape[1]
            weights_padding_cols = 0
            if scale_k % 4 != 0:
                padded_scale_k = round_up(scale_k, 4)
                pad_scale_k = padded_scale_k - scale_k
                scales = torch.nn.functional.pad(scales, (0, pad_scale_k, 0, 0))
                pad_weight_k = pad_scale_k * 8
                weight = torch.nn.functional.pad(weight, (0, pad_weight_k, 0, 0))
                weights_padding_cols = pad_weight_k

            epilogue_tile_m = 128
            shuffled_scale_shape = scales.shape
            if not weight.is_cuda:
                weight = weight.cuda()
            if scales.device != weight.device:
                scales = scales.to(device=weight.device)
            weight = flashinfer_ops.shuffle_matrix_a(
                weight.view(torch.uint8), epilogue_tile_m
            )
            scales = (
                flashinfer_ops.shuffle_matrix_sf_a(
                    scales.view(torch.uint8), epilogue_tile_m
                )
                .reshape(shuffled_scale_shape)
                .view(torch.float8_e4m3fn)
            )

            layer.weights_padding_cols = weights_padding_cols
            copy_or_rebind_param(layer, "weight", weight)
            copy_or_rebind_param(layer, "weight_scale_interleaved", scales)
            return
        weight, weights_padding_cols = pad_nvfp4_weight(w_swapped)
        layer.weights_padding_cols = weights_padding_cols
        copy_or_rebind_param(layer, "weight", weight)

        scales = layer.weight_scale
        scale_ndim = scales.ndim
        if scale_ndim == 2:
            scales = scales.unsqueeze(0)
        assert scales.ndim == 3
        B, M, K = scales.shape
        M_padded = round_up(M, 128)
        K_padded = round_up(K, 4)
        padded_scales = torch.zeros((B, M_padded, K_padded), dtype=scales.dtype)
        padded_scales[:B, :M, :K] = scales

        _, flashinfer_backend = _get_fp4_gemm_op()
        if flashinfer_backend is None:
            # CUTLASS (sgl_kernel) path: blockwise interleave to TMA layout
            padded_scales = padded_scales.reshape(
                B, M_padded // 128, 4, 32, K_padded // 4, 4
            )
            padded_scales = padded_scales.permute(0, 1, 4, 3, 2, 5)

        padded_scales = padded_scales.contiguous().cuda()
        padded_scales = (
            padded_scales.reshape(M_padded, K_padded)
            if scale_ndim == 2
            else padded_scales.reshape(B, M_padded, K_padded)
        )
        copy_or_rebind_param(layer, "weight_scale_interleaved", padded_scales)
```
**EN:** This block defines method `process_weights_after_loading` on `ModelOptFp4LinearMethod`. It processes weights after loading. Key calls include `layer.input_scale.max.to`, `layer.weight_scale_2.max.to`, `copy_or_rebind_param`, `_prepare_nvfp4_weight_bytes`, and `_get_fp4_gemm_op`. The implementation branches on conditions. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp4LinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `layer.input_scale.max.to`、`layer.weight_scale_2.max.to`、`copy_or_rebind_param`、`_prepare_nvfp4_weight_bytes` 和 `_get_fp4_gemm_op`。 实现中包含条件分支。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 572-631: `apply` implementation / `apply` 实现
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        output_dtype = x.dtype
        input_shape = x.shape
        x = x.view(-1, input_shape[-1])

        output_size = layer.output_size_per_partition
        output_shape = list(input_shape[:-1]) + [output_size]

        fp4_quantize = _get_fp4_quantize_op()
        if fp4_quantize is None:
            raise RuntimeError(
                "No FP4 quantization kernel available. Install flashinfer or sgl_kernel."
            )

        x_fp4, x_scale_interleaved = fp4_quantize(x, layer.input_scale_inv)
        weights_padding_cols = getattr(layer, "weights_padding_cols", 0)
        x_fp4 = pad_nvfp4_activation_for_cutlass(x_fp4, weights_padding_cols)

        w = layer.weight
        w_scale_interleaved = layer.weight_scale_interleaved

        if x_scale_interleaved.dtype == torch.uint8:
            x_scale_interleaved = x_scale_interleaved.view(torch.float8_e4m3fn)
        if w_scale_interleaved.dtype == torch.uint8:
            w_scale_interleaved = w_scale_interleaved.view(torch.float8_e4m3fn)
        fp4_gemm, flashinfer_backend = _get_fp4_gemm_op()
        if flashinfer_backend is not None:
            out = fp4_gemm(
                x_fp4,
                w.T,
                x_scale_interleaved,
                w_scale_interleaved.T,
                layer.alpha,
                output_dtype,
                backend=flashinfer_backend,
            )
        elif fp4_gemm is not None:
            out = fp4_gemm(
                x_fp4,
                w,
                x_scale_interleaved,
                w_scale_interleaved,
                layer.alpha,
                output_dtype,
            )
        else:
            raise RuntimeError(
                "No FP4 GEMM kernel available. Install flashinfer or sgl_kernel."
            )

        out = slice_nvfp4_output(out, output_size)

        if bias is not None:
            out = out + bias
        return out.view(*output_shape)
```
**EN:** This block defines method `apply` on `ModelOptFp4LinearMethod`. It applies function. Key calls include `x.view`, `_get_fp4_quantize_op`, `fp4_quantize`, `getattr`, and `pad_nvfp4_activation_for_cutlass`. The implementation branches on conditions. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp4LinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `x.view`、`_get_fp4_quantize_op`、`fp4_quantize`、`getattr` 和 `pad_nvfp4_activation_for_cutlass`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `_get_fp4_quantize_op`: Top-level function that retrieves fp4 quantize op. / 顶层函数，用于获取fp4 quantize op。
- `_get_fp4_gemm_op`: Top-level function that retrieves fp4 gemm op. / 顶层函数，用于获取fp4 gemm op。
- `_prepare_nvfp4_weight_bytes`: Normalize serialized NVFP4 bytes before padding for the runtime kernel. / 顶层函数，用于准备nvfp4 weight bytes。
- `_require_flashinfer`: Top-level function that handles require flashinfer logic. / 顶层函数，用于处理 require flashinfer 相关逻辑。
- `ModelOptQuantConfig`: Primary class that encapsulates model opt quant config behavior. / 核心类，用于封装 model opt quant config 相关行为。
- `ModelOptFp8Config`: Config class for ModelOpt FP8 diffusion checkpoints. / 核心类，用于封装 model opt fp8 config 相关行为。
- `ModelOptFp4Config`: Config class for NVFP4. / 核心类，用于封装 model opt fp4 config 相关行为。
- `ModelOptFp8LinearMethod`: Linear method for ModelOpt static FP8 checkpoints. / 核心类，用于封装 model opt fp8 linear method 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `re`, `functools`, `typing`
- **Third-party / 第三方依赖**: `torch`, `flashinfer`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.models.parameter`, `sglang.multimodal_gen.runtime.models.utils`, `sglang.multimodal_gen.runtime.platforms`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.modelopt_quant`, `sglang.srt.layers.quantization.utils`, `sglang.srt.layers.utils.common`, `sglang.srt.utils.common`

- **Total lines / 总行数**: 631
