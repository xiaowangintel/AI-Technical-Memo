# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This package initializer re-exports quantization components and registers the method mappings used by the SGLang SRT quantization stack. / 该包初始化文件重新导出量化组件，并注册 SGLang SRT 量化栈使用的方法映射。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://raw.githubusercontent.com/vllm-project/vllm/v0.5.5/vllm/model_executor/layers/quantization/__init__.py
from __future__ import annotations

import builtins
import inspect
from typing import TYPE_CHECKING, Dict, Optional, Type

import torch
```
**EN:** This block imports __future__, builtins, inspect, torch, sglang.srt.layers.moe.topk, sglang.srt.layers.quantization.auto_round, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, builtins, inspect, torch, sglang.srt.layers.moe.topk, sglang.srt.layers.quantization.auto_round, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config 等依赖，并为当前量化实现准备模块命名空间。

### Lines 14-14: class DummyConfig: definition
```python
class DummyConfig:
```
**EN:** This block declares `DummyConfig`, a configuration class for the quantization stack. It organizes behaviors such as override_quantization_method.
**CN:** 该代码块声明 `DummyConfig`，它是量化栈中的配置类，组织了 override_quantization_method 等行为。

### Lines 15-16: DummyConfig.override_quantization_method()
```python
    def override_quantization_method(self, *args, **kwargs):
        return None
```
**EN:** This block defines `DummyConfig.override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `DummyConfig.override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 19-19: initialize CompressedTensorsConfig
```python
CompressedTensorsConfig = DummyConfig
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as CompressedTensorsConfig.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 CompressedTensorsConfig。

### Lines 21-57: module imports and setup
```python
from sglang.srt.layers.quantization.auto_round import AutoRoundConfig
from sglang.srt.layers.quantization.awq import AWQConfig, AWQCPUConfig, AWQMarlinConfig
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.quantization.bitsandbytes import BitsAndBytesConfig
from sglang.srt.layers.quantization.blockwise_int8 import BlockInt8Config
from sglang.srt.layers.quantization.compressed_tensors.compressed_tensors import (
    CompressedTensorsConfig,
)
from sglang.srt.layers.quantization.fp8 import Fp8Config
from sglang.srt.layers.quantization.fpgemm_fp8 import FBGEMMFp8Config
from sglang.srt.layers.quantization.gguf import GGUFConfig
from sglang.srt.layers.quantization.gptq import GPTQConfig, GPTQMarlinConfig
from sglang.srt.layers.quantization.gptq_cpu import CPUGPTQConfig
from sglang.srt.layers.quantization.mlx import MlxQuantizationConfig
from sglang.srt.layers.quantization.modelopt_quant import (
    ModelOptFp4Config,
    ModelOptFp8Config,
    ModelOptMixedPrecisionConfig,
)
from sglang.srt.layers.quantization.modelslim.modelslim import ModelSlimConfig
from sglang.srt.layers.quantization.moe_wna16 import MoeWNA16Config
from sglang.srt.layers.quantization.mxfp4 import Mxfp4Config
from sglang.srt.layers.quantization.petit import PetitNvFp4Config
from sglang.srt.layers.quantization.qoq import QoQConfig
from sglang.srt.layers.quantization.quark.quark import QuarkConfig
from sglang.srt.layers.quantization.quark_int4fp8_moe import QuarkInt4Fp8Config
from sglang.srt.layers.quantization.w4afp8 import W4AFp8Config
from sglang.srt.layers.quantization.w8a8_fp8 import W8A8Fp8Config
from sglang.srt.layers.quantization.w8a8_int8 import W8A8Int8Config
from sglang.srt.utils import (
    cpu_has_amx_support,
    is_cuda,
    is_hip,
    is_mps,
    is_npu,
    mxfp_supported,
)
```
**EN:** This block imports __future__, builtins, inspect, torch, sglang.srt.layers.moe.topk, sglang.srt.layers.quantization.auto_round, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, builtins, inspect, torch, sglang.srt.layers.moe.topk, sglang.srt.layers.quantization.auto_round, sglang.srt.layers.quantization.awq, sglang.srt.layers.quantization.base_config 等依赖，并为当前量化实现准备模块命名空间。

### Lines 59-59: initialize _is_mxfp_supported
```python
_is_mxfp_supported = mxfp_supported()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_mxfp_supported.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_mxfp_supported。

### Lines 61-62: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.topk import TopKOutput
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 65-91: annotate BASE_QUANTIZATION_METHODS
```python
BASE_QUANTIZATION_METHODS: Dict[str, Type[QuantizationConfig]] = {
    "fp8": Fp8Config,
    "mxfp8": Fp8Config,
    "blockwise_int8": BlockInt8Config,
    "modelopt": ModelOptFp8Config,  # Auto-detect, defaults to FP8
    "modelopt_fp8": ModelOptFp8Config,
    "modelopt_fp4": ModelOptFp4Config,
    "modelopt_mixed": ModelOptMixedPrecisionConfig,
    "w8a8_int8": W8A8Int8Config,
    "w8a8_fp8": W8A8Fp8Config,
    "awq": AWQConfig,
    "awq_marlin": AWQMarlinConfig,
    "bitsandbytes": BitsAndBytesConfig,
    "gguf": GGUFConfig,
    "gptq": GPTQConfig,
    "gptq_marlin": GPTQMarlinConfig,
    "moe_wna16": MoeWNA16Config,
    "compressed-tensors": CompressedTensorsConfig,
    "qoq": QoQConfig,
    "w4afp8": W4AFp8Config,
    "petit_nvfp4": PetitNvFp4Config,
    "fbgemm_fp8": FBGEMMFp8Config,
    "quark": QuarkConfig,
    "auto-round": AutoRoundConfig,
    "modelslim": ModelSlimConfig,
    "quark_int4fp8_moe": QuarkInt4Fp8Config,
}
```
**EN:** This block declares and initializes the annotated symbol `BASE_QUANTIZATION_METHODS`.
**CN:** 该代码块声明并初始化带类型注解的符号 `BASE_QUANTIZATION_METHODS`。

### Lines 94-99: conditional logic for is_cuda() or (_is_mxfp_supported and is_hip())
```python
if is_cuda() or (_is_mxfp_supported and is_hip()):
    BASE_QUANTIZATION_METHODS.update(
        {
            "mxfp4": Mxfp4Config,
        }
    )
```
**EN:** This block applies conditional logic controlled by `is_cuda() or (_is_mxfp_supported and is_hip())`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `is_cuda() or (_is_mxfp_supported and is_hip())` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 102-108: conditional logic for is_mps()
```python
if is_mps():
    BASE_QUANTIZATION_METHODS.update(
        {
            "mlx_q4": MlxQuantizationConfig,
            "mlx_q8": MlxQuantizationConfig,
        }
    )
```
**EN:** This block applies conditional logic controlled by `is_mps()`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `is_mps()` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 111-117: initialize CPU_QUANTIZATION_METHODS
```python
CPU_QUANTIZATION_METHODS = {
    "fp8": Fp8Config,
    "w8a8_int8": W8A8Int8Config,
    "compressed-tensors": CompressedTensorsConfig,
    "awq": AWQCPUConfig,
    "gptq": CPUGPTQConfig,
}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as CPU_QUANTIZATION_METHODS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 CPU_QUANTIZATION_METHODS。

### Lines 119-119: initialize QUANTIZATION_METHODS
```python
QUANTIZATION_METHODS = {**BASE_QUANTIZATION_METHODS}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as QUANTIZATION_METHODS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 QUANTIZATION_METHODS。

### Lines 122-139: get_quantization_config()
```python
def get_quantization_config(quantization: str) -> Type[QuantizationConfig]:
    if quantization not in QUANTIZATION_METHODS:
        raise ValueError(
            f"Invalid quantization method: {quantization}. "
            f"Available methods: {list(QUANTIZATION_METHODS.keys())}"
        )
    from sglang.srt.utils import is_cpu

    if is_cpu() and cpu_has_amx_support():
        if quantization not in CPU_QUANTIZATION_METHODS:
            raise ValueError(
                f"Invalid quantization method on CPU: {quantization}. "
                f"Available methods on CPU: {list(QUANTIZATION_METHODS.keys())}"
            )
        else:
            return CPU_QUANTIZATION_METHODS[quantization]

    return QUANTIZATION_METHODS[quantization]
```
**EN:** This block defines `get_quantization_config()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `get_quantization_config()`，用于为调用方获取或计算派生值。

### Lines 142-142: initialize original_isinstance
```python
original_isinstance = builtins.isinstance
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as original_isinstance.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 original_isinstance。

## Key Concepts / 关键概念
- `DummyConfig`: A configuration class that structures file-level quantization behavior. / `DummyConfig` 是一个配置类，用于组织该文件中的量化行为。
- `CPU_QUANTIZATION_METHODS`: A module-level constant or registry. / `CPU_QUANTIZATION_METHODS`：模块级常量或注册表。
- `QUANTIZATION_METHODS`: A module-level constant or registry. / `QUANTIZATION_METHODS`：模块级常量或注册表。
- `get_quantization_config()` : A public function that retrieves or computes a derived value for callers. / `get_quantization_config()`：一个公开函数，用于为调用方获取或计算派生值。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `builtins`, `inspect`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.moe.topk`, `sglang.srt.layers.quantization.auto_round`, `sglang.srt.layers.quantization.awq`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.bitsandbytes`, `sglang.srt.layers.quantization.blockwise_int8`, `sglang.srt.layers.quantization.compressed_tensors.compressed_tensors`, `sglang.srt.layers.quantization.fp8`, `sglang.srt.layers.quantization.fpgemm_fp8`, `sglang.srt.layers.quantization.gguf`, `sglang.srt.layers.quantization.gptq`, `sglang.srt.layers.quantization.gptq_cpu`, `sglang.srt.layers.quantization.mlx`, `sglang.srt.layers.quantization.modelopt_quant`, `sglang.srt.layers.quantization.modelslim.modelslim`, `sglang.srt.layers.quantization.moe_wna16`
