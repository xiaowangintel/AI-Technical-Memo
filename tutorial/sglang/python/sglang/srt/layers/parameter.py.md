# parameter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/parameter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime layer support for the SGLang SRT runtime. It exposes symbols such as `_dtype_rank`, `copy_with_check`, `BasevLLMParameter`, and `_ColumnvLLMParameter` and connects them to backend-specific paths such as `NPU` and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了运行时层支持。它提供了 `_dtype_rank`、`copy_with_check`、`BasevLLMParameter` 以及 `_ColumnvLLMParameter` 等符号，并把这些符号连接到 `NPU` 和 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: File header and module overview
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 3-32: Imports, constants, and runtime setup
```python
"""Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/model_executor/parameter.py"""

import logging
from fractions import Fraction
from typing import Callable, Optional, Union

import torch
from torch.nn import Parameter

from sglang.srt.environ import envs
from sglang.srt.layers.utils import pad_or_narrow_weight
from sglang.srt.utils import is_cpu

__all__ = [
    "BasevLLMParameter",
    "PackedvLLMParameter",
    "PerTensorScaleParameter",
    "ModelWeightParameter",
    "ChannelQuantScaleParameter",
    "GroupQuantScaleParameter",
    "BlockQuantScaleParameter",
    "PackedColumnParameter",
    "RowvLLMParameter",
]

logger = logging.getLogger(__name__)

_is_cpu = is_cpu()
```
**EN:** This section prepares the module namespace. It imports `logging`, `fractions.Fraction`, `typing.Callable`, `typing.Optional`, `typing.Union`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `__all__`, `logger`, and `_is_cpu` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`fractions.Fraction`、`typing.Callable`、`typing.Optional`、`typing.Union` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `__all__`、`logger` 以及 `_is_cpu` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 33-50: Internal helper `_dtype_rank`
```python
def _dtype_rank(dtype: torch.dtype) -> Optional[int]:
    if dtype in (
        torch.float8_e4m3fn,
        torch.float8_e4m3fnuz,
        torch.float8_e5m2,
        torch.float8_e5m2fnuz,
        torch.float8_e8m0fnu,
    ):
        return 0
    if dtype in (torch.float16, torch.bfloat16):
        return 1
    if dtype == torch.float32:
        return 2
    if dtype == torch.float64:
        return 3
    return None
```
**EN:** This block defines `_dtype_rank` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_dtype_rank`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 51-81: Function `copy_with_check` and its core logic
```python
def copy_with_check(target: torch.Tensor, loaded_weight: torch.Tensor):
    """
    Copy `loaded_weight` into `target` while forbidding downcasts.
    bf16/fp16 share the same rank, and all fp8 variants share the same rank.
    """

    assert (
        target.shape == loaded_weight.shape
    ), f"{target.shape=}, {loaded_weight.shape=}"

    if target.dtype == loaded_weight.dtype:
        target.copy_(loaded_weight)
        return

    target_rank = _dtype_rank(target.dtype)
    loaded_rank = _dtype_rank(loaded_weight.dtype)

    if target_rank is None or loaded_rank is None:
        raise ValueError(
            f"Unsupported copy between dtypes: {target.dtype=}, {loaded_weight.dtype=}"
        )
    if target_rank < loaded_rank and not envs.SGLANG_QUANT_ALLOW_DOWNCASTING.get():
        raise ValueError(
            f"Downcasting not allowed: {target.dtype=}, {loaded_weight.dtype=}"
        )
    if loaded_rank == torch.float8_e8m0fnu:
        assert target_rank in {torch.float8_e8m0fnu, torch.float32}

    target.copy_(loaded_weight)
```
**EN:** This block defines `copy_with_check` and contains the main logic for this step. It mainly invokes `_dtype_rank`, `target.copy_`, `ValueError`, and `envs.SGLANG_QUANT_ALLOW_DOWNCASTING.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `target_rank` and `loaded_rank` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `copy_with_check`，并承载这一阶段的核心逻辑。 它主要调用 `_dtype_rank`、`target.copy_`、`ValueError` 以及 `envs.SGLANG_QUANT_ALLOW_DOWNCASTING.get`，说明该流程会编排底层辅助函数或计算内核。 像 `target_rank` 和 `loaded_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 82-88: Class `BasevLLMParameter` declaration and shared state
```python
class BasevLLMParameter(Parameter):
    """
    Base parameter for vLLM linear layers. Extends the torch.nn.parameter
    by taking in a linear weight loader. Will copy the loaded weight
    into the parameter when the provided weight loader is called.
    """
```
**EN:** This block introduces class `BasevLLMParameter` and the state shared by its methods. It inherits from `Parameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Base parameter for vLLM linear layers.
**CN:** 该代码块引入类 `BasevLLMParameter`，并定义其方法共享的状态。 它继承自 `Parameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 89-92: Internal helper `BasevLLMParameter.__new__`
```python
    def __new__(cls, data: torch.Tensor, **kwargs):

        return super().__new__(cls, data=data, requires_grad=False)
```
**EN:** This block defines `BasevLLMParameter.__new__` and contains the main logic for this step. It mainly invokes `super.__new__`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BasevLLMParameter.__new__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__new__`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 93-104: `BasevLLMParameter` initialization and state setup
```python
    def __init__(self, data: torch.Tensor, weight_loader: Callable):
        """
        Initialize the BasevLLMParameter

        :param data: torch tensor with the parameter data
        :param weight_loader: weight loader callable

        :returns: a torch.nn.parameter
        """

        self._weight_loader = weight_loader
```
**EN:** This block defines `BasevLLMParameter.__init__` and contains the main logic for this step. Intermediate names such as `self._weight_loader` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `BasevLLMParameter.__init__`，并承载这一阶段的核心逻辑。 像 `self._weight_loader` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 105-108: Function `BasevLLMParameter.weight_loader` and its core logic
```python
    @property
    def weight_loader(self):
        return self._weight_loader
```
**EN:** This block defines `BasevLLMParameter.weight_loader` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BasevLLMParameter.weight_loader`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 109-112: Internal helper `BasevLLMParameter._assert_and_load`
```python
    def _assert_and_load(self, loaded_weight: torch.Tensor):
        assert self.data.shape == loaded_weight.shape
        self.data.copy_(loaded_weight)
```
**EN:** This block defines `BasevLLMParameter._assert_and_load` and contains the main logic for this step. It mainly invokes `self.data.copy_`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `BasevLLMParameter._assert_and_load`，并承载这一阶段的核心逻辑。 它主要调用 `self.data.copy_`，说明该流程会编排底层辅助函数或计算内核。

### Lines 113-115: `BasevLLMParameter.load_column_parallel_weight` loader for column parallel weight
```python
    def load_column_parallel_weight(self, loaded_weight: torch.Tensor):
        self._assert_and_load(loaded_weight)
```
**EN:** This block defines `BasevLLMParameter.load_column_parallel_weight` and contains the main logic for this step. It mainly invokes `self._assert_and_load`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `BasevLLMParameter.load_column_parallel_weight`，并承载这一阶段的核心逻辑。 它主要调用 `self._assert_and_load`，说明该流程会编排底层辅助函数或计算内核。

### Lines 116-118: `BasevLLMParameter.load_row_parallel_weight` loader for row parallel weight
```python
    def load_row_parallel_weight(self, loaded_weight: torch.Tensor):
        self._assert_and_load(loaded_weight)
```
**EN:** This block defines `BasevLLMParameter.load_row_parallel_weight` and contains the main logic for this step. It mainly invokes `self._assert_and_load`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `BasevLLMParameter.load_row_parallel_weight`，并承载这一阶段的核心逻辑。 它主要调用 `self._assert_and_load`，说明该流程会编排底层辅助函数或计算内核。

### Lines 119-121: `BasevLLMParameter.load_merged_column_weight` loader for merged column weight
```python
    def load_merged_column_weight(self, loaded_weight: torch.Tensor, **kwargs):
        self._assert_and_load(loaded_weight)
```
**EN:** This block defines `BasevLLMParameter.load_merged_column_weight` and contains the main logic for this step. It mainly invokes `self._assert_and_load`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `BasevLLMParameter.load_merged_column_weight`，并承载这一阶段的核心逻辑。 它主要调用 `self._assert_and_load`，说明该流程会编排底层辅助函数或计算内核。

### Lines 122-125: `BasevLLMParameter.load_qkv_weight` loader for qkv weight
```python
    def load_qkv_weight(self, loaded_weight: torch.Tensor, **kwargs):
        self._assert_and_load(loaded_weight)
```
**EN:** This block defines `BasevLLMParameter.load_qkv_weight` and contains the main logic for this step. It mainly invokes `self._assert_and_load`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `BasevLLMParameter.load_qkv_weight`，并承载这一阶段的核心逻辑。 它主要调用 `self._assert_and_load`，说明该流程会编排底层辅助函数或计算内核。

### Lines 126-136: Class `_ColumnvLLMParameter` declaration and shared state
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
```
**EN:** This block introduces class `_ColumnvLLMParameter` and the state shared by its methods. It inherits from `BasevLLMParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Private class defining weight loading functionality (load_merged_column_weight, load_qkv_weight) for parameters being loaded into linear layers with column parallelism.
**CN:** 该代码块引入类 `_ColumnvLLMParameter`，并定义其方法共享的状态。 它继承自 `BasevLLMParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 137-140: `_ColumnvLLMParameter` initialization and state setup
```python
    def __init__(self, output_dim: int, **kwargs):
        self._output_dim = output_dim
        super().__init__(**kwargs)
```
**EN:** This block defines `_ColumnvLLMParameter.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._output_dim` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_ColumnvLLMParameter.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self._output_dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 141-144: Function `_ColumnvLLMParameter.output_dim` and its core logic
```python
    @property
    def output_dim(self):
        return self._output_dim
```
**EN:** This block defines `_ColumnvLLMParameter.output_dim` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_ColumnvLLMParameter.output_dim`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 145-176: `_ColumnvLLMParameter.load_column_parallel_weight` loader for column parallel weight
```python
    def load_column_parallel_weight(
        self,
        loaded_weight: torch.Tensor,
        tp_rank: int,
        use_presharded_weights: bool = False,
    ):
        if not use_presharded_weights:
            shard_size = self.data.shape[self.output_dim]

            from sglang.srt.model_loader.weight_utils import (
                narrow_padded_param_and_loaded_weight,
            )

            if _is_cpu:
                param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                    self.data,
                    loaded_weight,
                    0,  # param_data_start
                    tp_rank * shard_size,
                    self.output_dim,
                    shard_size,
                )
                assert param_data.shape == loaded_weight.shape
                param_data.copy_(loaded_weight)
                return
            else:
                loaded_weight = loaded_weight.narrow(
                    self.output_dim, tp_rank * shard_size, shard_size
                )

        copy_with_check(self.data, loaded_weight)
```
**EN:** This block defines `_ColumnvLLMParameter.load_column_parallel_weight` and contains the main logic for this step. It mainly invokes `copy_with_check`, `narrow_padded_param_and_loaded_weight`, `param_data.copy_`, and `loaded_weight.narrow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_size`, `param_data`, and `loaded_weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_ColumnvLLMParameter.load_column_parallel_weight`，并承载这一阶段的核心逻辑。 它主要调用 `copy_with_check`、`narrow_padded_param_and_loaded_weight`、`param_data.copy_` 以及 `loaded_weight.narrow`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_size`、`param_data` 以及 `loaded_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 177-225: `_ColumnvLLMParameter.load_merged_column_weight` loader for merged column weight
```python
    def load_merged_column_weight(self, loaded_weight: torch.Tensor, **kwargs):

        shard_offset = kwargs.get("shard_offset")
        shard_size = kwargs.get("shard_size")
        tp_rank = kwargs.get("tp_rank")
        use_presharded_weights = kwargs.get("use_presharded_weights")
        if (
            isinstance(self, (PackedColumnParameter, PackedvLLMParameter))
            and self.packed_dim == self.output_dim
        ):
            shard_size, shard_offset = self.adjust_shard_indexes_for_packing(
                shard_offset=shard_offset, shard_size=shard_size
            )

        param_data = self.data

        param_data = param_data.narrow(self.output_dim, shard_offset, shard_size)

        from sglang.srt.model_loader.weight_utils import (
            narrow_padded_param_and_loaded_weight,
        )

        if _is_cpu:
            param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                param_data,
                loaded_weight,
                0,  # param_data_start
                tp_rank * shard_size,
                self.output_dim,
                shard_size,
                not use_presharded_weights,
            )
        else:
            if not use_presharded_weights:
                # Padding for special case like qwen2_5_VL's mlp which is not 8-aligned
                start_idx = tp_rank * shard_size
                end_idx = start_idx + shard_size
                if end_idx > loaded_weight.shape[self.output_dim]:
                    loaded_weight = pad_or_narrow_weight(
                        loaded_weight, self.output_dim, start_idx, shard_size
                    )
                else:
                    loaded_weight = loaded_weight.narrow(
                        self.output_dim, start_idx, shard_size
                    )

        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines `_ColumnvLLMParameter.load_merged_column_weight` and contains the main logic for this step. It mainly invokes `kwargs.get`, `param_data.narrow`, `param_data.copy_`, `isinstance`, and `self.adjust_shard_indexes_for_packing`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_offset`, `shard_size`, `tp_rank`, `use_presharded_weights`, and `param_data` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_ColumnvLLMParameter.load_merged_column_weight`，并承载这一阶段的核心逻辑。 它主要调用 `kwargs.get`、`param_data.narrow`、`param_data.copy_`、`isinstance` 以及 `self.adjust_shard_indexes_for_packing`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_offset`、`shard_size`、`tp_rank`、`use_presharded_weights` 以及 `param_data` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 226-276: `_ColumnvLLMParameter.load_qkv_weight` loader for qkv weight
```python
    def load_qkv_weight(
        self,
        loaded_weight: torch.Tensor,
        tp_rank: int,
        use_presharded_weights: bool = False,
        **kwargs,
    ):

        shard_offset = kwargs.get("shard_offset")
        shard_size = kwargs.get("shard_size")
        shard_id = kwargs.get("shard_id")
        num_heads = kwargs.get("num_heads")

        if (
            isinstance(self, (PackedColumnParameter, PackedvLLMParameter))
            and self.output_dim == self.packed_dim
        ):
            shard_size, shard_offset = self.adjust_shard_indexes_for_packing(
                shard_offset=shard_offset, shard_size=shard_size
            )

        param_data = self.data
        shard_id = tp_rank if shard_id == "q" else tp_rank // num_heads
        param_data = param_data.narrow(self.output_dim, shard_offset, shard_size)

        if _is_cpu:
            from sglang.srt.model_loader.weight_utils import (
                narrow_padded_param_and_loaded_weight,
            )

            param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                param_data,
                loaded_weight,
                0,  # param_data_start
                shard_id * shard_size,
                self.output_dim,
                shard_size,
                not use_presharded_weights,
            )
        else:
            if not use_presharded_weights:
                loaded_weight = loaded_weight.narrow(
                    self.output_dim, shard_id * shard_size, shard_size
                )

        assert (
            param_data.shape == loaded_weight.shape
        ), f"{param_data.shape=}, {loaded_weight.shape=}"
        param_data.copy_(loaded_weight)
```
**EN:** This block defines `_ColumnvLLMParameter.load_qkv_weight` and contains the main logic for this step. It mainly invokes `kwargs.get`, `param_data.narrow`, `param_data.copy_`, `isinstance`, and `self.adjust_shard_indexes_for_packing`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_offset`, `shard_size`, `shard_id`, `num_heads`, and `param_data` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_ColumnvLLMParameter.load_qkv_weight`，并承载这一阶段的核心逻辑。 它主要调用 `kwargs.get`、`param_data.narrow`、`param_data.copy_`、`isinstance` 以及 `self.adjust_shard_indexes_for_packing`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_offset`、`shard_size`、`shard_id`、`num_heads` 以及 `param_data` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 277-284: Class `RowvLLMParameter` declaration and shared state
```python
class RowvLLMParameter(BasevLLMParameter):
    """
    Parameter class defining weight_loading functionality
    (load_row_parallel_weight) for parameters being loaded
    into linear layers with row parallel functionality.
    Requires an input_dim to be defined.
    """
```
**EN:** This block introduces class `RowvLLMParameter` and the state shared by its methods. It inherits from `BasevLLMParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameter class defining weight_loading functionality (load_row_parallel_weight) for parameters being loaded into linear layers with row parallel functionality.
**CN:** 该代码块引入类 `RowvLLMParameter`，并定义其方法共享的状态。 它继承自 `BasevLLMParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 285-288: `RowvLLMParameter` initialization and state setup
```python
    def __init__(self, input_dim: int, **kwargs):
        self._input_dim = input_dim
        super().__init__(**kwargs)
```
**EN:** This block defines `RowvLLMParameter.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._input_dim` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RowvLLMParameter.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self._input_dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 289-292: Function `RowvLLMParameter.input_dim` and its core logic
```python
    @property
    def input_dim(self):
        return self._input_dim
```
**EN:** This block defines `RowvLLMParameter.input_dim` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RowvLLMParameter.input_dim`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 293-339: `RowvLLMParameter.load_row_parallel_weight` loader for row parallel weight
```python
    def load_row_parallel_weight(
        self,
        loaded_weight: torch.Tensor,
        tp_rank: int,
        use_presharded_weights: bool = False,
    ):
        if not use_presharded_weights:
            shard_size = self.data.shape[self.input_dim]

            from sglang.srt.model_loader.weight_utils import (
                narrow_padded_param_and_loaded_weight,
            )

            if _is_cpu:
                param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                    self.data,
                    loaded_weight,
                    0,  # param_data_start
                    tp_rank * shard_size,
                    self.input_dim,
                    shard_size,
                )

                assert param_data.shape == loaded_weight.shape
                param_data.copy_(loaded_weight)

                return
            else:
                # Padding for special case like qwen2_5_VL's mlp which is not 8-aligned
                start_idx = tp_rank * shard_size
                end_idx = start_idx + shard_size
                if end_idx > loaded_weight.shape[self.input_dim]:
                    loaded_weight = pad_or_narrow_weight(
                        loaded_weight, self.input_dim, start_idx, shard_size
                    )
                else:
                    loaded_weight = loaded_weight.narrow(
                        self.input_dim, start_idx, shard_size
                    )

        if len(loaded_weight.shape) == 0:
            loaded_weight = loaded_weight.reshape(1)

        assert self.data.shape == loaded_weight.shape
        self.data.copy_(loaded_weight)
```
**EN:** This block defines `RowvLLMParameter.load_row_parallel_weight` and contains the main logic for this step. It mainly invokes `self.data.copy_`, `len`, `loaded_weight.reshape`, `narrow_padded_param_and_loaded_weight`, and `param_data.copy_`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_size`, `loaded_weight`, `param_data`, `start_idx`, and `end_idx` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RowvLLMParameter.load_row_parallel_weight`，并承载这一阶段的核心逻辑。 它主要调用 `self.data.copy_`、`len`、`loaded_weight.reshape`、`narrow_padded_param_and_loaded_weight` 以及 `param_data.copy_`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_size`、`loaded_weight`、`param_data`、`start_idx` 以及 `end_idx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 340-348: Class `ModelWeightParameter` declaration and shared state
```python
class ModelWeightParameter(_ColumnvLLMParameter, RowvLLMParameter):
    """
    Parameter class for linear layer weights. Uses both column and
    row parallelism.
    """

    pass
```
**EN:** This block introduces class `ModelWeightParameter` and the state shared by its methods. It inherits from `_ColumnvLLMParameter` and `RowvLLMParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameter class for linear layer weights.
**CN:** 该代码块引入类 `ModelWeightParameter`，并定义其方法共享的状态。 它继承自 `_ColumnvLLMParameter` 和 `RowvLLMParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 349-357: Class `GroupQuantScaleParameter` declaration and shared state
```python
class GroupQuantScaleParameter(_ColumnvLLMParameter, RowvLLMParameter):
    """
    Parameter class for weight scales loaded for weights with
    grouped quantization. Uses both column and row parallelism.
    """

    pass
```
**EN:** This block introduces class `GroupQuantScaleParameter` and the state shared by its methods. It inherits from `_ColumnvLLMParameter` and `RowvLLMParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameter class for weight scales loaded for weights with grouped quantization.
**CN:** 该代码块引入类 `GroupQuantScaleParameter`，并定义其方法共享的状态。 它继承自 `_ColumnvLLMParameter` 和 `RowvLLMParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 358-366: Class `ChannelQuantScaleParameter` declaration and shared state
```python
class ChannelQuantScaleParameter(_ColumnvLLMParameter):
    """
    Parameter class for weight scales loaded for weights with
    channel-wise quantization. Equivalent to _ColumnvLLMParameter.
    """

    pass
```
**EN:** This block introduces class `ChannelQuantScaleParameter` and the state shared by its methods. It inherits from `_ColumnvLLMParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameter class for weight scales loaded for weights with channel-wise quantization.
**CN:** 该代码块引入类 `ChannelQuantScaleParameter`，并定义其方法共享的状态。 它继承自 `_ColumnvLLMParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 367-375: Class `BlockQuantScaleParameter` declaration and shared state
```python
class BlockQuantScaleParameter(_ColumnvLLMParameter, RowvLLMParameter):
    """
    Parameter class for weight scales loaded for weights with
    block-wise quantization. Uses both column and row parallelism.
    """

    pass
```
**EN:** This block introduces class `BlockQuantScaleParameter` and the state shared by its methods. It inherits from `_ColumnvLLMParameter` and `RowvLLMParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameter class for weight scales loaded for weights with block-wise quantization.
**CN:** 该代码块引入类 `BlockQuantScaleParameter`，并定义其方法共享的状态。 它继承自 `_ColumnvLLMParameter` 和 `RowvLLMParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 376-389: Class `PerTensorScaleParameter` declaration and shared state
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
```
**EN:** This block introduces class `PerTensorScaleParameter` and the state shared by its methods. It inherits from `BasevLLMParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameter class for scales where the number of scales is equivalent to the number of logical matrices in fused linear layers (e.g.
**CN:** 该代码块引入类 `PerTensorScaleParameter`，并定义其方法共享的状态。 它继承自 `BasevLLMParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 390-393: `PerTensorScaleParameter` initialization and state setup
```python
    def __init__(self, **kwargs):
        self.qkv_idxs = {"q": 0, "k": 1, "v": 2}
        super().__init__(**kwargs)
```
**EN:** This block defines `PerTensorScaleParameter.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.qkv_idxs` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `PerTensorScaleParameter.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.qkv_idxs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 394-405: Internal helper `PerTensorScaleParameter._shard_id_as_int`
```python
    def _shard_id_as_int(self, shard_id: Union[str, int]) -> int:
        if isinstance(shard_id, int):
            return shard_id

        # if not int, assume shard_id for qkv
        # map to int and return
        assert isinstance(shard_id, str)
        assert shard_id in self.qkv_idxs
        return self.qkv_idxs[shard_id]

    # For row parallel layers, no sharding needed
    # load weight into parameter as is
```
**EN:** This block defines `PerTensorScaleParameter._shard_id_as_int` and contains the main logic for this step. It mainly invokes `isinstance`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PerTensorScaleParameter._shard_id_as_int`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 406-410: `PerTensorScaleParameter.load_row_parallel_weight` loader for row parallel weight
```python
    def load_row_parallel_weight(self, *args, **kwargs):
        kwargs.pop("tp_rank", None)
        kwargs.pop("use_presharded_weights", None)
        super().load_row_parallel_weight(*args, **kwargs)
```
**EN:** This block defines `PerTensorScaleParameter.load_row_parallel_weight` and contains the main logic for this step. It mainly invokes `kwargs.pop` and `super.load_row_parallel_weight`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `PerTensorScaleParameter.load_row_parallel_weight`，并承载这一阶段的核心逻辑。 它主要调用 `kwargs.pop` 和 `super.load_row_parallel_weight`，说明该流程会编排底层辅助函数或计算内核。

### Lines 411-413: `PerTensorScaleParameter.load_merged_column_weight` loader for merged column weight
```python
    def load_merged_column_weight(self, *args, **kwargs):
        self._load_into_shard_id(*args, **kwargs)
```
**EN:** This block defines `PerTensorScaleParameter.load_merged_column_weight` and contains the main logic for this step. It mainly invokes `self._load_into_shard_id`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `PerTensorScaleParameter.load_merged_column_weight`，并承载这一阶段的核心逻辑。 它主要调用 `self._load_into_shard_id`，说明该流程会编排底层辅助函数或计算内核。

### Lines 414-416: `PerTensorScaleParameter.load_qkv_weight` loader for qkv weight
```python
    def load_qkv_weight(self, *args, **kwargs):
        self._load_into_shard_id(*args, **kwargs)
```
**EN:** This block defines `PerTensorScaleParameter.load_qkv_weight` and contains the main logic for this step. It mainly invokes `self._load_into_shard_id`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `PerTensorScaleParameter.load_qkv_weight`，并承载这一阶段的核心逻辑。 它主要调用 `self._load_into_shard_id`，说明该流程会编排底层辅助函数或计算内核。

### Lines 417-421: `PerTensorScaleParameter.load_column_parallel_weight` loader for column parallel weight
```python
    def load_column_parallel_weight(self, *args, **kwargs):
        kwargs.pop("tp_rank", None)
        kwargs.pop("use_presharded_weights", None)
        super().load_row_parallel_weight(*args, **kwargs)
```
**EN:** This block defines `PerTensorScaleParameter.load_column_parallel_weight` and contains the main logic for this step. It mainly invokes `kwargs.pop` and `super.load_row_parallel_weight`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `PerTensorScaleParameter.load_column_parallel_weight`，并承载这一阶段的核心逻辑。 它主要调用 `kwargs.pop` 和 `super.load_row_parallel_weight`，说明该流程会编排底层辅助函数或计算内核。

### Lines 422-443: Internal helper `PerTensorScaleParameter._load_into_shard_id`
```python
    def _load_into_shard_id(
        self, loaded_weight: torch.Tensor, shard_id: Union[str, int], **kwargs
    ):
        """
        Slice the parameter data based on the shard id for
        loading.
        """

        param_data = self.data
        shard_id = self._shard_id_as_int(shard_id)

        # AutoFP8 scales do not have a shape
        # compressed-tensors scales do have a shape
        if len(loaded_weight.shape) != 0:
            assert loaded_weight.shape[0] == 1
            loaded_weight = loaded_weight[0]

        param_data = param_data[shard_id]
        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines `PerTensorScaleParameter._load_into_shard_id` and contains the main logic for this step. It mainly invokes `self._shard_id_as_int`, `param_data.copy_`, and `len`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `param_data`, `shard_id`, and `loaded_weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `PerTensorScaleParameter._load_into_shard_id`，并承载这一阶段的核心逻辑。 它主要调用 `self._shard_id_as_int`、`param_data.copy_` 以及 `len`，说明该流程会编排底层辅助函数或计算内核。 像 `param_data`、`shard_id` 以及 `loaded_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 444-450: Class `PackedColumnParameter` declaration and shared state
```python
class PackedColumnParameter(_ColumnvLLMParameter):
    """
    Parameter for model parameters which are packed on disk
    and support column parallelism only. See PackedvLLMParameter
    for more details on the packed properties.
    """
```
**EN:** This block introduces class `PackedColumnParameter` and the state shared by its methods. It inherits from `_ColumnvLLMParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameter for model parameters which are packed on disk and support column parallelism only.
**CN:** 该代码块引入类 `PackedColumnParameter`，并定义其方法共享的状态。 它继承自 `_ColumnvLLMParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 451-462: `PackedColumnParameter` initialization and state setup
```python
    def __init__(
        self,
        packed_factor: Union[int, Fraction],
        packed_dim: int,
        marlin_tile_size: Optional[int] = None,
        **kwargs,
    ):
        self._packed_factor = packed_factor
        self._packed_dim = packed_dim
        self._marlin_tile_size = marlin_tile_size
        super().__init__(**kwargs)
```
**EN:** This block defines `PackedColumnParameter.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._packed_factor`, `self._packed_dim`, and `self._marlin_tile_size` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `PackedColumnParameter.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self._packed_factor`、`self._packed_dim` 以及 `self._marlin_tile_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 463-466: Function `PackedColumnParameter.packed_dim` and its core logic
```python
    @property
    def packed_dim(self):
        return self._packed_dim
```
**EN:** This block defines `PackedColumnParameter.packed_dim` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PackedColumnParameter.packed_dim`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 467-470: Function `PackedColumnParameter.packed_factor` and its core logic
```python
    @property
    def packed_factor(self):
        return self._packed_factor
```
**EN:** This block defines `PackedColumnParameter.packed_factor` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PackedColumnParameter.packed_factor`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 471-474: Function `PackedColumnParameter.marlin_tile_size` and its core logic
```python
    @property
    def marlin_tile_size(self):
        return self._marlin_tile_size
```
**EN:** This block defines `PackedColumnParameter.marlin_tile_size` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PackedColumnParameter.marlin_tile_size`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 475-483: Function `PackedColumnParameter.adjust_shard_indexes_for_packing` and its core logic
```python
    def adjust_shard_indexes_for_packing(self, shard_size, shard_offset):
        return _adjust_shard_indexes_for_packing(
            shard_size=shard_size,
            shard_offset=shard_offset,
            packed_factor=self.packed_factor,
            marlin_tile_size=self.marlin_tile_size,
        )
```
**EN:** This block defines `PackedColumnParameter.adjust_shard_indexes_for_packing` and contains the main logic for this step. It mainly invokes `_adjust_shard_indexes_for_packing`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PackedColumnParameter.adjust_shard_indexes_for_packing`，并承载这一阶段的核心逻辑。 它主要调用 `_adjust_shard_indexes_for_packing`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 484-494: Class `PackedvLLMParameter` declaration and shared state
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
```
**EN:** This block introduces class `PackedvLLMParameter` and the state shared by its methods. It inherits from `ModelWeightParameter`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameter for model weights which are packed on disk.
**CN:** 该代码块引入类 `PackedvLLMParameter`，并定义其方法共享的状态。 它继承自 `ModelWeightParameter`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 495-506: `PackedvLLMParameter` initialization and state setup
```python
    def __init__(
        self,
        packed_factor: Union[int, Fraction],
        packed_dim: int,
        marlin_tile_size: Optional[int] = None,
        **kwargs,
    ):
        self._packed_factor = packed_factor
        self._packed_dim = packed_dim
        self._marlin_tile_size = marlin_tile_size
        super().__init__(**kwargs)
```
**EN:** This block defines `PackedvLLMParameter.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._packed_factor`, `self._packed_dim`, and `self._marlin_tile_size` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `PackedvLLMParameter.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self._packed_factor`、`self._packed_dim` 以及 `self._marlin_tile_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 507-510: Function `PackedvLLMParameter.packed_dim` and its core logic
```python
    @property
    def packed_dim(self):
        return self._packed_dim
```
**EN:** This block defines `PackedvLLMParameter.packed_dim` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PackedvLLMParameter.packed_dim`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 511-514: Function `PackedvLLMParameter.packed_factor` and its core logic
```python
    @property
    def packed_factor(self):
        return self._packed_factor
```
**EN:** This block defines `PackedvLLMParameter.packed_factor` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PackedvLLMParameter.packed_factor`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 515-518: Function `PackedvLLMParameter.marlin_tile_size` and its core logic
```python
    @property
    def marlin_tile_size(self):
        return self._marlin_tile_size
```
**EN:** This block defines `PackedvLLMParameter.marlin_tile_size` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PackedvLLMParameter.marlin_tile_size`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 519-527: Function `PackedvLLMParameter.adjust_shard_indexes_for_packing` and its core logic
```python
    def adjust_shard_indexes_for_packing(self, shard_size, shard_offset):
        return _adjust_shard_indexes_for_packing(
            shard_size=shard_size,
            shard_offset=shard_offset,
            packed_factor=self.packed_factor,
            marlin_tile_size=self.marlin_tile_size,
        )
```
**EN:** This block defines `PackedvLLMParameter.adjust_shard_indexes_for_packing` and contains the main logic for this step. It mainly invokes `_adjust_shard_indexes_for_packing`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `PackedvLLMParameter.adjust_shard_indexes_for_packing`，并承载这一阶段的核心逻辑。 它主要调用 `_adjust_shard_indexes_for_packing`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 528-585: Function `permute_param_layout_` and its core logic
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

    curr_input_dim = getattr(param, "input_dim", None)
    curr_output_dim = getattr(param, "output_dim", None)

    if curr_input_dim is None or curr_output_dim is None:
        assert param.data.dim() == 2, (
            "permute_param_layout_ only supports 2D parameters when either "
            "input_dim or output_dim is not set"
        )

    # if one of the dimensions is not set, set it to the opposite of the other
    #  we can only do this since we asserted the parameter is 2D above
    if curr_input_dim is None:
        assert curr_output_dim is not None, "either input or output dim must be set"
        curr_input_dim = (curr_output_dim + 1) % 2
    if curr_output_dim is None:
        assert curr_input_dim is not None, "either input or output dim must be set"
        curr_output_dim = (curr_input_dim + 1) % 2

    # create permutation from the current layout to the layout with
    # self.input_dim at input_dim and self.output_dim at output_dim preserving
    # other dimensions
    perm = [
        i for i in range(param.data.dim()) if i not in [curr_input_dim, curr_output_dim]
    ]
    perm.insert(input_dim, curr_input_dim)
    perm.insert(output_dim, curr_output_dim)

    if "packed_dim" in kwargs:
        assert (
            hasattr(param, "packed_dim")
            and param.packed_dim == perm[kwargs["packed_dim"]]
        ), "permute_param_layout_ currently doesn't support repacking"

    param.data = param.data.permute(*perm)
    if hasattr(param, "_input_dim"):
        param._input_dim = input_dim
    if hasattr(param, "_output_dim"):
        param._output_dim = output_dim
    if "packed_dim" in kwargs and hasattr(param, "_packed_dim"):
        param._packed_dim = kwargs["packed_dim"]

    return param
```
**EN:** This block defines `permute_param_layout_` and contains the main logic for this step. It mainly invokes `getattr`, `perm.insert`, `param.data.permute`, `hasattr`, and `param.data.dim`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `curr_input_dim`, `curr_output_dim`, and `perm` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `permute_param_layout_`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`、`perm.insert`、`param.data.permute`、`hasattr` 以及 `param.data.dim`，说明该流程会编排底层辅助函数或计算内核。 像 `curr_input_dim`、`curr_output_dim` 以及 `perm` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 586-589: Internal helper `_adjust_shard_indexes_for_marlin`
```python
def _adjust_shard_indexes_for_marlin(shard_size, shard_offset, marlin_tile_size):
    return shard_size * marlin_tile_size, shard_offset * marlin_tile_size
```
**EN:** This block defines `_adjust_shard_indexes_for_marlin` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_adjust_shard_indexes_for_marlin`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 590-601: Internal helper `_adjust_shard_indexes_for_packing`
```python
def _adjust_shard_indexes_for_packing(
    shard_size, shard_offset, packed_factor, marlin_tile_size
):
    shard_size = shard_size // packed_factor
    shard_offset = shard_offset // packed_factor
    if marlin_tile_size is not None:
        return _adjust_shard_indexes_for_marlin(
            shard_size=shard_size,
            shard_offset=shard_offset,
            marlin_tile_size=marlin_tile_size,
        )
    return shard_size, shard_offset
```
**EN:** This block defines `_adjust_shard_indexes_for_packing` and contains the main logic for this step. It mainly invokes `_adjust_shard_indexes_for_marlin`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_size` and `shard_offset` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_adjust_shard_indexes_for_packing`，并承载这一阶段的核心逻辑。 它主要调用 `_adjust_shard_indexes_for_marlin`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_size` 和 `shard_offset` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_dtype_rank`, `copy_with_check`, `BasevLLMParameter`, `_ColumnvLLMParameter`, and `RowvLLMParameter`. / **主要符号**：核心入口包括 `_dtype_rank`、`copy_with_check`、`BasevLLMParameter`、`_ColumnvLLMParameter` 以及 `RowvLLMParameter`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Standard library**: `logging`, `fractions.Fraction`, `typing.Callable`, `typing.Optional`, and `typing.Union` / **标准库**：`logging`、`fractions.Fraction`、`typing.Callable`、`typing.Optional` 以及 `typing.Union`
- **Third-party**: `torch` and `torch.nn.Parameter` / **第三方依赖**：`torch` 和 `torch.nn.Parameter`
- **Internal SGLang modules**: `sglang.srt.environ.envs`, `sglang.srt.layers.utils.pad_or_narrow_weight`, `sglang.srt.utils.is_cpu`, and `sglang.srt.model_loader.weight_utils.narrow_padded_param_and_loaded_weight` / **SGLang 内部模块**：`sglang.srt.environ.envs`、`sglang.srt.layers.utils.pad_or_narrow_weight`、`sglang.srt.utils.is_cpu` 以及 `sglang.srt.model_loader.weight_utils.narrow_padded_param_and_loaded_weight`
