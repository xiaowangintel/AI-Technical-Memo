# parameter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/parameter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for parameter within the multimodal runtime. Key symbols include `BasevLLMParameter`, `_ColumnvLLMParameter`, `RowvLLMParameter`. / 该模块实现多模态运行时中与 parameter 相关的模型构件。 关键符号包括 `BasevLLMParameter`, `_ColumnvLLMParameter`, `RowvLLMParameter`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/model_executor/parameter.py

from collections.abc import Callable
from fractions import Fraction
from typing import Any

import torch
from torch.nn import Parameter

from sglang.multimodal_gen.runtime.distributed import get_tp_rank
from sglang.multimodal_gen.runtime.models.utils import _make_synced_weight_loader
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 21-82: Class `BasevLLMParameter` / 类 `BasevLLMParameter`
```python
class BasevLLMParameter(Parameter):
    """
    Base parameter for vLLM linear layers. Extends the torch.nn.parameter
    by taking in a linear weight loader. Will copy the loaded weight
    into the parameter when the provided weight loader is called.
    """

    def __new__(cls, data: torch.Tensor, **kwargs):

        return super().__new__(cls, data=data, requires_grad=False)

    def __init__(self, data: torch.Tensor, weight_loader: Callable):
        """
        Initialize the BasevLLMParameter
# ...
        self._assert_and_load(loaded_weight)

    def load_qkv_weight(self, loaded_weight: torch.Tensor, **kwargs) -> None:
        self._assert_and_load(loaded_weight)
```
**EN:** This class models `BasevLLMParameter` as a specialization of `Parameter`. Base parameter for vLLM linear layers. Important methods include `__new__`, `__init__`, `weight_loader`, `_is_1d_and_scalar`.
**CN:** 该类实现 `BasevLLMParameter`，并继承/扩展 `Parameter`。 文档字符串指出：Base parameter for vLLM linear layers. 其中较重要的方法包括 `__new__`, `__init__`, `weight_loader`, `_is_1d_and_scalar`。

### Lines 85-166: Class `_ColumnvLLMParameter` / 类 `_ColumnvLLMParameter`
```python
class _ColumnvLLMParameter(BasevLLMParameter):
    """
    Private class defining weight loading functionality
    (load_merged_column_weight, load_qkv_weight)
    for parameters being loaded into linear layers with column
    parallelism. This includes QKV and MLP layers which are
    not already fused on disk. Requires an output dimension
    to be defined. Called within the weight loader of
    each of the column parallel linear layers.
    """

    def __init__(self, output_dim: int, **kwargs):
        self._output_dim = output_dim
        super().__init__(**kwargs)
# ...
        )

        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This class models `_ColumnvLLMParameter` as a specialization of `BasevLLMParameter`. Private class defining weight loading functionality Important methods include `__init__`, `output_dim`, `load_column_parallel_weight`, `load_merged_column_weight`.
**CN:** 该类实现 `_ColumnvLLMParameter`，并继承/扩展 `BasevLLMParameter`。 文档字符串指出：Private class defining weight loading functionality 其中较重要的方法包括 `__init__`, `output_dim`, `load_column_parallel_weight`, `load_merged_column_weight`。

### Lines 169-196: Class `RowvLLMParameter` / 类 `RowvLLMParameter`
```python
class RowvLLMParameter(BasevLLMParameter):
    """
    Parameter class defining weight_loading functionality
    (load_row_parallel_weight) for parameters being loaded
    into linear layers with row parallel functionality.
    Requires an input_dim to be defined.
    """

    def __init__(self, input_dim: int, **kwargs):
        self._input_dim = input_dim
        super().__init__(**kwargs)

    @property
    def input_dim(self):
# ...
            loaded_weight = loaded_weight.reshape(1)

        assert self.data.shape == loaded_weight.shape
        self.data.copy_(loaded_weight)
```
**EN:** This class models `RowvLLMParameter` as a specialization of `BasevLLMParameter`. Parameter class defining weight_loading functionality Important methods include `__init__`, `input_dim`, `load_row_parallel_weight`.
**CN:** 该类实现 `RowvLLMParameter`，并继承/扩展 `BasevLLMParameter`。 文档字符串指出：Parameter class defining weight_loading functionality 其中较重要的方法包括 `__init__`, `input_dim`, `load_row_parallel_weight`。

### Lines 199-205: Class `ModelWeightParameter` / 类 `ModelWeightParameter`
```python
class ModelWeightParameter(_ColumnvLLMParameter, RowvLLMParameter):
    """
    Parameter class for linear layer weights. Uses both column and
    row parallelism.
    """

    pass
```
**EN:** This class models `ModelWeightParameter` as a specialization of `_ColumnvLLMParameter`, `RowvLLMParameter`. Parameter class for linear layer weights.
**CN:** 该类实现 `ModelWeightParameter`，并继承/扩展 `_ColumnvLLMParameter`, `RowvLLMParameter`。 文档字符串指出：Parameter class for linear layer weights.

### Lines 208-214: Class `GroupQuantScaleParameter` / 类 `GroupQuantScaleParameter`
```python
class GroupQuantScaleParameter(_ColumnvLLMParameter, RowvLLMParameter):
    """
    Parameter class for weight scales loaded for weights with
    grouped quantization. Uses both column and row parallelism.
    """

    pass
```
**EN:** This class models `GroupQuantScaleParameter` as a specialization of `_ColumnvLLMParameter`, `RowvLLMParameter`. Parameter class for weight scales loaded for weights with
**CN:** 该类实现 `GroupQuantScaleParameter`，并继承/扩展 `_ColumnvLLMParameter`, `RowvLLMParameter`。 文档字符串指出：Parameter class for weight scales loaded for weights with

### Lines 217-223: Class `ChannelQuantScaleParameter` / 类 `ChannelQuantScaleParameter`
```python
class ChannelQuantScaleParameter(_ColumnvLLMParameter):
    """
    Parameter class for weight scales loaded for weights with
    channel-wise quantization. Equivalent to _ColumnvLLMParameter.
    """

    pass
```
**EN:** This class models `ChannelQuantScaleParameter` as a specialization of `_ColumnvLLMParameter`. Parameter class for weight scales loaded for weights with
**CN:** 该类实现 `ChannelQuantScaleParameter`，并继承/扩展 `_ColumnvLLMParameter`。 文档字符串指出：Parameter class for weight scales loaded for weights with

### Lines 226-287: Class `PerTensorScaleParameter` / 类 `PerTensorScaleParameter`
```python
class PerTensorScaleParameter(BasevLLMParameter):
    """
    Parameter class for scales where the number of scales is
    equivalent to the number of logical matrices in fused linear
    layers (e.g. for QKV, there are 3 scales loaded from disk).
    This is relevant to weights with per-tensor quantization.
    Adds functionality to map the scalers to a shard during
    weight loading.

    Note: additional parameter manipulation may be handled
    for each quantization config specifically, within
    process_weights_after_loading
    """

# ...

        param_data = param_data[shard_id]
        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This class models `PerTensorScaleParameter` as a specialization of `BasevLLMParameter`. Parameter class for scales where the number of scales is Important methods include `__init__`, `_shard_id_as_int`, `load_row_parallel_weight`, `load_merged_column_weight`.
**CN:** 该类实现 `PerTensorScaleParameter`，并继承/扩展 `BasevLLMParameter`。 文档字符串指出：Parameter class for scales where the number of scales is 其中较重要的方法包括 `__init__`, `_shard_id_as_int`, `load_row_parallel_weight`, `load_merged_column_weight`。

### Lines 290-317: Class `PackedColumnParameter` / 类 `PackedColumnParameter`
```python
class PackedColumnParameter(_ColumnvLLMParameter):
    """
    Parameter for model parameters which are packed on disk
    and support column parallelism only. See PackedvLLMParameter
    for more details on the packed properties.
    """

    def __init__(self, packed_factor: int | Fraction, packed_dim: int, **kwargs):
        self._packed_factor = packed_factor
        self._packed_dim = packed_dim
        super().__init__(**kwargs)

    @property
    def packed_dim(self):
# ...
            shard_size=shard_size,
            shard_offset=shard_offset,
            packed_factor=self.packed_factor,
        )
```
**EN:** This class models `PackedColumnParameter` as a specialization of `_ColumnvLLMParameter`. Parameter for model parameters which are packed on disk Important methods include `__init__`, `packed_dim`, `packed_factor`, `adjust_shard_indexes_for_packing`.
**CN:** 该类实现 `PackedColumnParameter`，并继承/扩展 `_ColumnvLLMParameter`。 文档字符串指出：Parameter for model parameters which are packed on disk 其中较重要的方法包括 `__init__`, `packed_dim`, `packed_factor`, `adjust_shard_indexes_for_packing`。

### Lines 320-349: Class `PackedvLLMParameter` / 类 `PackedvLLMParameter`
```python
class PackedvLLMParameter(ModelWeightParameter):
    """
    Parameter for model weights which are packed on disk.
    Example: GPTQ Marlin weights are int4 or int8, packed into int32.
    Extends the ModelWeightParameter to take in the
    packed factor, the packed dimension, and optionally, marlin
    tile size for marlin kernels. Adjusts the shard_size and
    shard_offset for fused linear layers model weight loading
    by accounting for packing and optionally, marlin tile size.
    """

    def __init__(self, packed_factor: int | Fraction, packed_dim: int, **kwargs):
        self._packed_factor = packed_factor
        self._packed_dim = packed_dim
# ...
            shard_size=shard_size,
            shard_offset=shard_offset,
            packed_factor=self.packed_factor,
        )
```
**EN:** This class models `PackedvLLMParameter` as a specialization of `ModelWeightParameter`. Parameter for model weights which are packed on disk. Important methods include `__init__`, `packed_dim`, `packed_factor`, `adjust_shard_indexes_for_packing`.
**CN:** 该类实现 `PackedvLLMParameter`，并继承/扩展 `ModelWeightParameter`。 文档字符串指出：Parameter for model weights which are packed on disk. 其中较重要的方法包括 `__init__`, `packed_dim`, `packed_factor`, `adjust_shard_indexes_for_packing`。

### Lines 352-358: Class `BlockQuantScaleParameter` / 类 `BlockQuantScaleParameter`
```python
class BlockQuantScaleParameter(_ColumnvLLMParameter, RowvLLMParameter):
    """
    Parameter class for weight scales loaded for weights with
    block-wise quantization. Uses both column and row parallelism.
    """

    pass
```
**EN:** This class models `BlockQuantScaleParameter` as a specialization of `_ColumnvLLMParameter`, `RowvLLMParameter`. Parameter class for weight scales loaded for weights with
**CN:** 该类实现 `BlockQuantScaleParameter`，并继承/扩展 `_ColumnvLLMParameter`, `RowvLLMParameter`。 文档字符串指出：Parameter class for weight scales loaded for weights with

### Lines 361-416: Function `permute_param_layout_` / 函数 `permute_param_layout_`
```python
def permute_param_layout_(
    param: BasevLLMParameter, input_dim: int, output_dim: int, **kwargs
) -> BasevLLMParameter:
    """
    Permute a parameter's layout to the specified input and output dimensions,
    useful for forcing the parameter into a known layout, for example, if I need
    a packed (quantized) weight matrix to be in the layout
        {input_dim = 0, output_dim = 1, packed_dim = 0}
    then I can call:
        permute_param_layout_(x, input_dim=0, output_dim=1, packed_dim=0)
    to ensure x is in the correct layout (permuting it to the correct layout if
    required, asserting if it cannot get it to the correct layout)
    """

# ...
    if "packed_dim" in kwargs and hasattr(param, "_packed_dim"):
        param._packed_dim = kwargs["packed_dim"]

    return param
```
**EN:** This function drives `permute_param_layout_` with inputs such as `param`, `input_dim`, `output_dim`. Permute a parameter's layout to the specified input and output dimensions,
**CN:** 这个函数负责 `permute_param_layout_`，主要处理 `param`, `input_dim`, `output_dim` 等输入。 文档字符串说明：Permute a parameter's layout to the specified input and output dimensions,

### Lines 419-424: Function `_adjust_shard_indexes_for_packing` / 函数 `_adjust_shard_indexes_for_packing`
```python
def _adjust_shard_indexes_for_packing(
    shard_size, shard_offset, packed_factor
) -> tuple[Any, Any]:
    shard_size = shard_size // packed_factor
    shard_offset = shard_offset // packed_factor
    return shard_size, shard_offset
```
**EN:** This function drives `_adjust_shard_indexes_for_packing` with inputs such as `shard_size`, `shard_offset`, `packed_factor`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_adjust_shard_indexes_for_packing`，主要处理 `shard_size`, `shard_offset`, `packed_factor` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Artifact storage management / 产物存储管理
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.models.utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `fractions`, `torch`, `torch.nn`
- **Stdlib / 标准库**: `collections.abc`, `typing`
