# auto_round.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/auto_round.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for auto round quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 自动 舍入 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0

import logging
import re
from fractions import Fraction
from typing import Any, Optional, Union

import torch
```
**EN:** This block imports fractions, logging, re, torch, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 fractions, logging, re, torch, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config 等依赖，并为当前量化实现准备模块命名空间。

### Lines 10-10: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 12-12: module imports and setup
```python
from sglang.srt.layers.quantization.utils import get_scalar_types
```
**EN:** This block imports fractions, logging, re, torch, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 fractions, logging, re, torch, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config 等依赖，并为当前量化实现准备模块命名空间。

### Lines 14-14: initialize (ScalarType, scalar_types)
```python
ScalarType, scalar_types = get_scalar_types()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as (ScalarType, scalar_types).
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 (ScalarType, scalar_types)。

### Lines 16-17: module imports and setup
```python
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.utils import is_npu
```
**EN:** This block imports fractions, logging, re, torch, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 fractions, logging, re, torch, sglang.srt.layers.linear, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config 等依赖，并为当前量化实现准备模块命名空间。

### Lines 19-19: initialize _is_npu
```python
_is_npu = is_npu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_npu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_npu。

### Lines 22-26: class AutoRoundConfig: definition
```python
class AutoRoundConfig(QuantizationConfig):
    """Config class for AutoRound.
    Reference: https://arxiv.org/pdf/2309.05516
    """
```
**EN:** This block declares `AutoRoundConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_name, get_supported_act_dtypes.
**CN:** 该代码块声明 `AutoRoundConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_name, get_supported_act_dtypes 等行为。

### Lines 27-27: AutoRoundConfig member: initialize SUPPORTED_BITS
```python
    SUPPORTED_BITS = {2, 3, 4, 8}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as SUPPORTED_BITS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 SUPPORTED_BITS。

### Lines 28-28: AutoRoundConfig member: initialize SUPPORTED_DTYPES
```python
    SUPPORTED_DTYPES = {"int"}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as SUPPORTED_DTYPES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 SUPPORTED_DTYPES。

### Lines 29-29: AutoRoundConfig member: initialize SUPPORTED_FORMATS
```python
    SUPPORTED_FORMATS = {"auto_round:auto_gptq", "auto_round:auto_awq"}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as SUPPORTED_FORMATS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 SUPPORTED_FORMATS。

### Lines 30-30: AutoRoundConfig member: initialize SUPPORTED_BACKENDS
```python
    SUPPORTED_BACKENDS = {"auto", "gptq", "gptq:marlin", "awq", "awq:marlin", "marlin"}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as SUPPORTED_BACKENDS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 SUPPORTED_BACKENDS。

### Lines 32-77: AutoRoundConfig.__init__()
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        sym: bool = True,
        packing_format: str = "auto_round:auto_gptq",
        block_name_to_quantize: Optional[Union[str, list[str]]] = None,
        extra_config: Optional[dict[str, Any]] = None,
        data_type: str = "int",
        backend: str = "auto",
    ) -> None:
        super().__init__()
        if weight_bits not in self.SUPPORTED_BITS:
            raise ValueError(
                f"Unsupported weight_bits: {weight_bits}, "
                f"currently only support  {self.SUPPORTED_BITS}"
            )
        if data_type not in self.SUPPORTED_DTYPES:
            raise ValueError(
                f"Unsupported data_type: {data_type},"
                f" currently only support  {self.SUPPORTED_DTYPES}"
            )
        if packing_format not in self.SUPPORTED_FORMATS:
            raise ValueError(
                f"Unsupported packing_format: {packing_format}, "
                f"currently only support  {self.SUPPORTED_FORMATS}"
            )
        if backend not in self.SUPPORTED_BACKENDS:
            raise ValueError(
                f"Unsupported backend: {backend},  "
                f"currently only support  {self.SUPPORTED_BACKENDS}"
            )

        self.weight_bits = weight_bits
        self.group_size = group_size
        self.sym = sym
        self.packing_format = packing_format
        self.block_name_to_quantize = (
            block_name_to_quantize.split(",")
            if isinstance(block_name_to_quantize, str)
            else block_name_to_quantize
        )
        self.extra_config = extra_config
        self.data_type = data_type
        self.backend = backend
        self.pack_factor = Fraction(32, weight_bits)
```
**EN:** This block defines `AutoRoundConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AutoRoundConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 79-83: AutoRoundConfig.__repr__()
```python
    def __repr__(self) -> str:
        return (
            f"AutoRoundConfig(weight_bits={self.weight_bits}, "
            f"group_size={self.group_size}, sym={self.sym})"
        )
```
**EN:** This block defines `AutoRoundConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `AutoRoundConfig.__repr__()`，用于构造可读的调试表示。

### Lines 85-87: AutoRoundConfig.get_name()
```python
    @classmethod
    def get_name(cls):
        return "auto-round"
```
**EN:** This block defines `AutoRoundConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AutoRoundConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 89-91: AutoRoundConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.half, torch.bfloat16]
```
**EN:** This block defines `AutoRoundConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AutoRoundConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 93-95: AutoRoundConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 60
```
**EN:** This block defines `AutoRoundConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AutoRoundConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 97-99: AutoRoundConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return ["quantization_config.json"]
```
**EN:** This block defines `AutoRoundConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AutoRoundConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 101-120: AutoRoundConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: dict[str, Any]) -> "AutoRoundConfig":
        return cls(
            weight_bits=cls.get_from_keys(config, ["bits"]),
            group_size=cls.get_from_keys(config, ["group_size"]),
            sym=cls.get_from_keys(config, ["sym"]),
            packing_format=cls.get_from_keys_or(
                config,
                ["packing_format"],
                "auto_round:auto_gptq",
            ),
            block_name_to_quantize=cls.get_from_keys_or(
                config, ["block_name_to_quantize", "to_quant_block_names"], None
            ),
            extra_config=cls.get_from_keys_or(config, ["extra_config"], None),
            data_type=cls.get_from_keys_or(config, ["data_type"], "int"),
            backend=cls.get_from_keys_or(
                config, ["backend", "vllm_backend", "sglang_backend"], "auto"
            ),
        )
```
**EN:** This block defines `AutoRoundConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `AutoRoundConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 122-127: AutoRoundConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> list[str]:
        """Returns the activation function names that should be post-scaled.

        For now, this is only used by AWQ.
        """
        raise NotImplementedError
```
**EN:** This block defines `AutoRoundConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AutoRoundConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 129-173: AutoRoundConfig.get_layer_config() (part 1/2)
```python
    def get_layer_config(self, layer, layer_name: str):
        from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead

        def get_config(name: str, quantized: bool = True):
            if not self.extra_config:
                return (
                    self.weight_bits if quantized else 16,
                    self.group_size if quantized else -1,
                    self.sym if quantized else True,
                )

            # Exact match first
            if name in self.extra_config:
                cfg = self.extra_config[name]
                return (
                    cfg.get("bits", self.weight_bits if quantized else 16),
                    cfg.get("group_size", self.group_size if quantized else -1),
                    cfg.get("sym", self.sym if quantized else True),
                )

            REGEX_SPECIAL_CHARS = set(r"*+?^$()[]{}|\\")
            for pattern, cfg in self.extra_config.items():
                if not isinstance(pattern, str) or not any(
                    c in REGEX_SPECIAL_CHARS for c in pattern
                ):
                    continue

                try:
                    if re.fullmatch(pattern, name):
                        return (
                            cfg.get("bits", self.weight_bits if quantized else 16),
                            cfg.get("group_size", self.group_size if quantized else -1),
                            cfg.get("sym", self.sym if quantized else True),
                        )
                except re.error:
                    # Invalid regex, ignore.
                    continue

            return (
                self.weight_bits if quantized else 16,
                self.group_size if quantized else -1,
                self.sym if quantized else True,
            )

        # 1. Exact match from config
```
**EN:** This segment of `AutoRoundConfig.get_layer_config()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `AutoRoundConfig.get_layer_config()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 174-215: AutoRoundConfig.get_layer_config() (part 2/2)
```python
        if self.extra_config and layer_name in self.extra_config:
            return get_config(layer_name)

        # 2. Determine whether layer should be quantized
        quantized = not isinstance(layer, ParallelLMHead)
        if self.block_name_to_quantize:
            quantized = any(
                layer_name.startswith(name) for name in self.block_name_to_quantize
            )

        # 3. Handle fused MoE
        if self.extra_config and "fusedmoe" in layer.__class__.__name__.lower():
            moe_configs = [
                get_config(name, quantized)
                for name in self.extra_config
                if name.startswith(layer_name)
            ]
            if moe_configs:
                if len(set(moe_configs)) == 1:
                    return moe_configs[0]
                raise ValueError(
                    f"Fused MoE layer '{layer_name}' requires "
                    f"consistent quant config for all sub-layers"
                )

        # 4. Handle fused QKV or other patterns
        if self.extra_config:
            for fusion_key, sub_keys in self.packed_modules_mapping.items():
                if fusion_key in layer_name and layer_name.count(fusion_key) == 1:
                    sub_names = [
                        layer_name.replace(fusion_key, sub_key) for sub_key in sub_keys
                    ]
                    sub_configs = [get_config(name, quantized) for name in sub_names]
                    if len(set(sub_configs)) == 1:
                        return sub_configs[0]
                    raise ValueError(
                        f"Fused module '{layer_name}' requires "
                        f"consistent quant config for {sub_names}"
                    )

        # 5. Fallback or try a regular expression match
        return get_config(layer_name, quantized)
```
**EN:** This segment of `AutoRoundConfig.get_layer_config()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `AutoRoundConfig.get_layer_config()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 217-218: AutoRoundConfig.check_quantized()
```python
    def check_quantized(self, weight_bits: int) -> bool:
        return weight_bits < 16
```
**EN:** This block defines `AutoRoundConfig.check_quantized()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `AutoRoundConfig.check_quantized()`，用于处理张量或权重量化逻辑。

### Lines 220-264: AutoRoundConfig.apply_awq_quant_layer() (part 1/2)
```python
    def apply_awq_quant_layer(self, layer, prefix: str, backend: str = "auto"):
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE
        from sglang.srt.layers.quantization.marlin_utils import (
            check_marlin_supported,
            check_moe_marlin_supports_layer,
        )
        from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
        from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead

        weight_bits, group_size, sym = self.get_layer_config(layer, prefix)
        if not self.check_quantized(weight_bits):
            if isinstance(layer, (LinearBase, ParallelLMHead)):
                return UnquantizedLinearMethod()
            else:
                return None
        logger.debug(
            "[%s] Type: %s, Bits: %s, Group Size: %s, Sym: %s",
            prefix,
            layer.__class__.__name__,
            weight_bits,
            group_size,
            sym,
        )
        if backend == "auto" or "marlin" in backend:
            AWQ_TYPE_MAP = {
                4: scalar_types.uint4,
                8: scalar_types.uint8,
            }
            use_marlin = (weight_bits in AWQ_TYPE_MAP) and check_marlin_supported(
                AWQ_TYPE_MAP[weight_bits], group_size, not sym
            )
            if isinstance(layer, FusedMoE):
                use_marlin = use_marlin and check_moe_marlin_supports_layer(
                    layer, group_size
                )

        else:
            use_marlin = False
        if use_marlin:
            from sglang.srt.layers.quantization.awq import (
                AWQLinearMethod,
                AWQMarlinConfig,
                AWQMoEMethod,
            )
```
**EN:** This segment of `AutoRoundConfig.apply_awq_quant_layer()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `AutoRoundConfig.apply_awq_quant_layer()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 265-305: AutoRoundConfig.apply_awq_quant_layer() (part 2/2)
```python

            quant_args_marlin = AWQMarlinConfig(
                weight_bits=weight_bits,
                group_size=group_size,
                zero_point=not sym,
                lm_head_quantized=False,
                full_config={},
                modules_to_not_convert=[],
            )
        else:
            from sglang.srt.layers.quantization.awq import AWQConfig, AWQLinearMethod

            quant_args = AWQConfig(
                weight_bits=weight_bits,
                group_size=group_size,
                zero_point=not sym,
            )

        if isinstance(layer, FusedMoE):
            if use_marlin:
                layer.scheme = quant_args_marlin.get_moe_scheme(layer)
                return AWQMoEMethod(quant_args_marlin)
            from sglang.srt.layers.quantization.moe_wna16 import MoeWNA16Config

            config = {
                "quant_method": "awq",
                "bits": weight_bits,
                "group_size": group_size,
                "zero_point": not sym,
                "lm_head": False,
            }
            return MoeWNA16Config.from_config(config).get_quant_method(layer, prefix)

        if isinstance(layer, (LinearBase, ParallelLMHead)):
            if use_marlin:
                layer.scheme = quant_args_marlin.get_linear_scheme(layer)
                return AWQLinearMethod(quant_args_marlin)
            else:
                layer.scheme = quant_args.get_linear_scheme(layer)
                return AWQLinearMethod(quant_args)
        return None
```
**EN:** This segment of `AutoRoundConfig.apply_awq_quant_layer()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `AutoRoundConfig.apply_awq_quant_layer()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 307-351: AutoRoundConfig.apply_gptq_quant_layer() (part 1/3)
```python
    def apply_gptq_quant_layer(self, layer, prefix: str, backend: str = "auto"):
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE
        from sglang.srt.layers.quantization.gptq import (
            GPTQConfig,
            GPTQLinearAscendMethod,
            GPTQMoEAscendMethod,
        )
        from sglang.srt.layers.quantization.marlin_utils import (
            check_marlin_supported,
            check_moe_marlin_supports_layer,
        )
        from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
        from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead

        weight_bits, group_size, sym = self.get_layer_config(layer, prefix)
        if not self.check_quantized(weight_bits):
            if isinstance(layer, (LinearBase, ParallelLMHead)):
                return UnquantizedLinearMethod()
            else:
                return None

        logger.debug(
            "[%s] Type: %s, Bits: %s, Group Size: %s, Sym: %s",
            prefix,
            layer.__class__.__name__,
            weight_bits,
            group_size,
            sym,
        )
        if _is_npu:
            quant_args = GPTQConfig(
                weight_bits=weight_bits,
                group_size=group_size,
                lm_head_quantized=False,
                desc_act=False,
                dynamic={},
            )
            quant_args.sym = sym

            if isinstance(layer, FusedMoE):
                return GPTQMoEAscendMethod(quant_args)

            if isinstance(layer, (LinearBase, ParallelLMHead)):
                return GPTQLinearAscendMethod(quant_args)
```
**EN:** This segment of `AutoRoundConfig.apply_gptq_quant_layer()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `AutoRoundConfig.apply_gptq_quant_layer()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 352-396: AutoRoundConfig.apply_gptq_quant_layer() (part 2/3)
```python

            return None

        if backend == "auto" or "marlin" in backend:
            GPTQ_TYPE_MAP = {
                (4, True): scalar_types.uint4b8,
                (8, True): scalar_types.uint8b128,
            }
            use_marlin = (weight_bits, sym) in GPTQ_TYPE_MAP and check_marlin_supported(
                GPTQ_TYPE_MAP[(weight_bits, sym)], group_size, has_zp=not sym
            )
            if isinstance(layer, FusedMoE):
                use_marlin = use_marlin and check_moe_marlin_supports_layer(
                    layer, group_size
                )
        else:
            use_marlin = False
        if use_marlin:
            from sglang.srt.layers.quantization.gptq import (
                GPTQMarlinConfig,
                GPTQMarlinLinearMethod,
                GPTQMarlinMoEMethod,
            )

            quant_args_marlin = GPTQMarlinConfig(
                weight_bits=weight_bits,
                group_size=group_size,
                is_sym=sym,
                lm_head_quantized=False,
                desc_act=False,
                dynamic={},
                full_config={},
            )
        else:
            from sglang.srt.layers.quantization.gptq import GPTQConfig, GPTQLinearMethod

            quant_args = GPTQConfig(
                weight_bits=weight_bits,
                group_size=group_size,
                lm_head_quantized=False,
                desc_act=False,
                dynamic={},
            )

        if isinstance(layer, FusedMoE):
```
**EN:** This segment of `AutoRoundConfig.apply_gptq_quant_layer()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `AutoRoundConfig.apply_gptq_quant_layer()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 397-418: AutoRoundConfig.apply_gptq_quant_layer() (part 3/3)
```python
            if use_marlin:
                from sglang.srt.layers.quantization.moe_wna16 import MoeWNA16Config

                config = {
                    "quant_method": "gptq",
                    "bits": weight_bits,
                    "group_size": group_size,
                    "sym": sym,
                    "lm_head": False,
                }
                return MoeWNA16Config.from_config(config).get_quant_method(
                    layer, prefix
                )
            return GPTQMarlinMoEMethod(quant_args_marlin)

        if isinstance(layer, (LinearBase, ParallelLMHead)):
            if use_marlin:
                return GPTQMarlinLinearMethod(quant_args_marlin)
            else:
                return GPTQLinearMethod(quant_args)

        return None
```
**EN:** This segment of `AutoRoundConfig.apply_gptq_quant_layer()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `AutoRoundConfig.apply_gptq_quant_layer()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 420-425: AutoRoundConfig.get_quant_method()
```python
    def get_quant_method(self, layer: torch.nn.Module, prefix: str):
        # TODO enable CPU quant method later
        if "gptq" in self.packing_format or "gptq" in self.backend:
            return self.apply_gptq_quant_layer(layer, prefix)
        if "awq" in self.packing_format or "awq" in self.backend:
            return self.apply_awq_quant_layer(layer, prefix)
```
**EN:** This block defines `AutoRoundConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AutoRoundConfig.get_quant_method()`，用于为调用方获取或计算派生值。

## Key Concepts / 关键概念
- `AutoRoundConfig`: A configuration class that structures file-level quantization behavior. / `AutoRoundConfig` 是一个配置类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `fractions`, `logging`, `re`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.quantization.awq`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.gptq`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.layers.quantization.moe_wna16`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.layers.quantization.utils`, `sglang.srt.layers.vocab_parallel_embedding`, `sglang.srt.utils`
