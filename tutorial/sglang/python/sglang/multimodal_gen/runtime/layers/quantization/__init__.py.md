# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `register_quantization_config`, and `get_quantization_config`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `register_quantization_config` 和 `get_quantization_config` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-18: module setup and imports / 模块初始化与导入
```python
from typing import Literal, get_args

from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
)
from sglang.multimodal_gen.runtime.layers.quantization.fp8 import Fp8Config
from sglang.multimodal_gen.runtime.layers.quantization.modelopt_fp8 import (
    ModelOptFp8Config as ModelOptFp8DiffusionConfig,
)
from sglang.multimodal_gen.runtime.layers.quantization.modelopt_quant import (
    ModelOptFp4Config,
    ModelOptFp8Config,
)
from sglang.multimodal_gen.runtime.layers.quantization.modelslim import ModelSlimConfig
from sglang.multimodal_gen.runtime.layers.quantization.mxfp4 import Mxfp4Config
from sglang.multimodal_gen.runtime.layers.quantization.mxfp8_npu import MXFP8Config
```
**EN:** This block establishes the module context and imports `typing`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.quantization.fp8`, `sglang.multimodal_gen.runtime.layers.quantization.modelopt_fp8`, `sglang.multimodal_gen.runtime.layers.quantization.modelopt_quant`, and `sglang.multimodal_gen.runtime.layers.quantization.modelslim`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `typing`、`sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`、`sglang.multimodal_gen.runtime.layers.quantization.fp8`、`sglang.multimodal_gen.runtime.layers.quantization.modelopt_fp8`、`sglang.multimodal_gen.runtime.layers.quantization.modelopt_quant` 和 `sglang.multimodal_gen.runtime.layers.quantization.modelslim`。这些依赖为后续实现提供所需符号。

### Lines 20-35: supporting statements / 辅助语句
```python
QuantizationMethods = Literal[
    "fp8", "modelopt", "modelopt_fp8", "modelopt_fp4", "modelslim", "mxfp4"
]

QUANTIZATION_METHODS: list[str] = list(get_args(QuantizationMethods))

# The customized quantization methods which will be added to this dict.
_CUSTOMIZED_METHOD_TO_QUANT_CONFIG = {
    "modelopt": ModelOptFp8DiffusionConfig,
    "modelopt_fp8": ModelOptFp8Config,
    "modelopt_fp4": ModelOptFp4Config,
    "modelslim": ModelSlimConfig,
    "fp8": Fp8Config,
    "mxfp4": Mxfp4Config,
    "mxfp8": MXFP8Config,
}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `QuantizationMethods`, `QUANTIZATION_METHODS`, and `_CUSTOMIZED_METHOD_TO_QUANT_CONFIG`. The code collaborates with `list`, and `get_args`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `QuantizationMethods`、`QUANTIZATION_METHODS` 和 `_CUSTOMIZED_METHOD_TO_QUANT_CONFIG` 等名称。 代码会与 `list` 和 `get_args` 协同工作。

### Lines 38-63: `register_quantization_config` implementation / `register_quantization_config` 实现
```python
def register_quantization_config(quantization: str):
    """Register a customized vllm quantization config.

    When a quantization method is not supported by vllm, you can register a customized
    quantization config to support it.

    Args:
        quantization (str): The quantization method name.


    """  # noqa: E501

    def _wrapper(quant_config_cls):
        if quantization in QUANTIZATION_METHODS:
            raise ValueError(
                f"The quantization method `{quantization}` is already exists."
            )
        if not issubclass(quant_config_cls, QuantizationConfig):
            raise ValueError(
                "The quantization config must be a subclass of " "`QuantizationConfig`."
            )
        _CUSTOMIZED_METHOD_TO_QUANT_CONFIG[quantization] = quant_config_cls
        QUANTIZATION_METHODS.append(quantization)
        return quant_config_cls

    return _wrapper
```
**EN:** This block defines function `register_quantization_config`. Register a customized vllm quantization config. When a quantization method is not supported by vllm, you can register a customized quantization config to support it. Key calls include `QUANTIZATION_METHODS.append`, `ValueError`, and `issubclass`. The implementation branches on conditions. Parameters such as `quantization` drive the behavior in this section.
**CN:** 该代码块定义了函数 `register_quantization_config`。 它用于注册quantization config。 关键调用包括 `QUANTIZATION_METHODS.append`、`ValueError` 和 `issubclass`。 实现中包含条件分支。 本段逻辑主要由 `quantization` 等参数驱动。

### Lines 66-74: `get_quantization_config` implementation / `get_quantization_config` 实现
```python
def get_quantization_config(quantization: str) -> type[QuantizationConfig]:
    if quantization not in QUANTIZATION_METHODS:
        raise ValueError(f"Invalid quantization method: {quantization}")

    method_to_config: dict[str, type[QuantizationConfig]] = {}
    # Update the `method_to_config` with customized quantization methods.
    method_to_config.update(_CUSTOMIZED_METHOD_TO_QUANT_CONFIG)

    return method_to_config[quantization]
```
**EN:** This block defines function `get_quantization_config`. It retrieves quantization config. Key calls include `method_to_config.update`, and `ValueError`. The implementation branches on conditions. Parameters such as `quantization` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_quantization_config`。 它用于获取quantization config。 关键调用包括 `method_to_config.update` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `quantization` 等参数驱动。

### Lines 77-82: supporting statements / 辅助语句
```python
__all__ = [
    "QuantizationMethods",
    "QuantizationConfig",
    "get_quantization_config",
    "QUANTIZATION_METHODS",
]
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `__all__`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `__all__` 等名称。

## Key Concepts / 关键概念
- `register_quantization_config`: Register a customized vllm quantization config. / 顶层函数，用于注册quantization config。
- `get_quantization_config`: Top-level function that retrieves quantization config. / 顶层函数，用于获取quantization config。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.quantization.fp8`, `sglang.multimodal_gen.runtime.layers.quantization.modelopt_fp8`, `sglang.multimodal_gen.runtime.layers.quantization.modelopt_quant`, `sglang.multimodal_gen.runtime.layers.quantization.modelslim`, `sglang.multimodal_gen.runtime.layers.quantization.mxfp4`, `sglang.multimodal_gen.runtime.layers.quantization.mxfp8_npu`

- **Total lines / 总行数**: 82
