# linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/linear.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `adjust_marlin_shard`, `adjust_bitsandbytes_4bit_shard`, `adjust_scalar_to_fused_array`, and `adjust_shard_offsets` and connects them to backend-specific paths such as `NPU` and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成。它提供了 `adjust_marlin_shard`、`adjust_bitsandbytes_4bit_shard`、`adjust_scalar_to_fused_array` 以及 `adjust_shard_offsets` 等符号，并把这些符号连接到 `NPU` 和 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: File header and module overview
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 3-85: Imports, conditional backend setup, and runtime guards
```python
"""Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/model_executor/layers/linear.py"""

from __future__ import annotations

import itertools
import logging
from typing import TYPE_CHECKING, Dict, List, Optional, Tuple

import torch
from torch import nn
from torch.nn.parameter import Parameter, UninitializedParameter

from sglang.kernel_api_logging import wrap_method_with_debug_kernel_once
from sglang.srt.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    get_tp_group,
    split_tensor_along_last_dim,
    tensor_model_parallel_all_gather,
    tensor_model_parallel_all_reduce,
    tensor_model_parallel_quant_all_reduce,
)
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.layers.dp_attention import (
    get_attention_tp_group,
    is_allocation_symmetric,
)
from sglang.srt.layers.parameter import (
    BasevLLMParameter,
    BlockQuantScaleParameter,
    PackedColumnParameter,
    PackedvLLMParameter,
    PerTensorScaleParameter,
    RowvLLMParameter,
    _ColumnvLLMParameter,
)
from sglang.srt.layers.utils import pad_or_narrow_weight
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import get_bool_env_var, is_cpu, is_hip, is_npu, set_weight_attrs

if TYPE_CHECKING:
    from sglang.srt.layers.quantization.base_config import (
        QuantizationConfig,
        QuantizeMethodBase,
    )

_is_hip = is_hip()
_disable_hip_linear_quant = _is_hip and get_bool_env_var(
    "SGLANG_ROCM_DISABLE_LINEARQUANT"
)

logger = logging.getLogger(__name__)

WEIGHT_LOADER_V2_SUPPORTED = [
    "CompressedTensorsLinearMethod",
    "AWQLinearMethod",
    "GPTQMarlinLinearMethod",
    "Fp8LinearMethod",
    "BlockInt8LinearMethod",
    "MarlinLinearMethod",
    "QQQLinearMethod",
    "GPTQMarlin24LinearMethod",
    "TPUInt8LinearMethod",
    "GPTQLinearMethod",
    "FBGEMMFp8LinearMethod",
    "GPTQLinearAscendMethod",
    "GPTQLinearIntelAMXMethod",
    "GPTQMoEAscendMethod",
    "GPTQMoEIntelAMXMethod",
    "ModelOptFp8LinearMethod",
    "ModelOptFp4LinearMethod",
    "IPEXAWQLinearMethod",
    "PetitNvFp4LinearMethod",
    "QuarkInt4Fp8LinearMethod",
]

_is_cpu = is_cpu()
_is_npu = is_npu()
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `itertools`, `logging`, `typing.TYPE_CHECKING`, `typing.Dict`, and `typing.List`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip`, `_disable_hip_linear_quant`, `logger`, `WEIGHT_LOADER_V2_SUPPORTED`, and `_is_cpu` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`itertools`、`logging`、`typing.TYPE_CHECKING`、`typing.Dict` 以及 `typing.List`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip`、`_disable_hip_linear_quant`、`logger`、`WEIGHT_LOADER_V2_SUPPORTED` 以及 `_is_cpu` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 86-93: Function `adjust_marlin_shard` and its core logic
```python
def adjust_marlin_shard(param, shard_size, shard_offset):
    marlin_tile_size = getattr(param, "marlin_tile_size", None)
    if marlin_tile_size is None:
        return shard_size, shard_offset

    return shard_size * marlin_tile_size, shard_offset * marlin_tile_size
```
**EN:** This block defines `adjust_marlin_shard` and contains the main logic for this step. It mainly invokes `getattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `marlin_tile_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `adjust_marlin_shard`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`，说明该流程会编排底层辅助函数或计算内核。 像 `marlin_tile_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 94-108: Function `adjust_bitsandbytes_4bit_shard` and its core logic
```python
def adjust_bitsandbytes_4bit_shard(
    param: Parameter, shard_offsets: Dict[str, Tuple[int, int]], loaded_shard_id: str
) -> Tuple[int, int]:
    """Adjust the quantization offsets and sizes for BitsAndBytes sharding."""

    total, _ = shard_offsets["total"]
    orig_offset, orig_size = shard_offsets[loaded_shard_id]

    quantized_total = param.data.shape[0]
    quantized_offset = orig_offset * quantized_total // total
    quantized_size = orig_size * quantized_total // total

    return quantized_size, quantized_offset
```
**EN:** This block defines `adjust_bitsandbytes_4bit_shard` and contains the main logic for this step. Intermediate names such as `total`, `_`, `orig_offset`, `orig_size`, and `quantized_total` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `adjust_bitsandbytes_4bit_shard`，并承载这一阶段的核心逻辑。 像 `total`、`_`、`orig_offset`、`orig_size` 以及 `quantized_total` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 109-131: Function `adjust_scalar_to_fused_array` and its core logic
```python
def adjust_scalar_to_fused_array(param, loaded_weight, shard_id):
    """For fused modules (QKV and MLP) we have an array of length
    N that holds 1 scale for each "logical" matrix. So the param
    is an array of length N. The loaded_weight corresponds to
    one of the shards on disk. Here, we slice the param based on
    the shard_id for loading.
    """
    qkv_idxs = {"q": 0, "k": 1, "v": 2}

    if isinstance(shard_id, str):
        shard_id = qkv_idxs[shard_id]
    elif not isinstance(shard_id, int):
        raise ValueError(f"Unknown Shard Id {shard_id}")

    # AutoFP8 scales do not have a shape
    # compressed-tensors scales do have a shape
    if len(loaded_weight.shape) != 0:
        assert loaded_weight.shape[0] == 1
        loaded_weight = loaded_weight[0]

    return param[shard_id], loaded_weight
```
**EN:** This block defines `adjust_scalar_to_fused_array` and contains the main logic for this step. It mainly invokes `isinstance`, `len`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `qkv_idxs`, `shard_id`, and `loaded_weight` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `adjust_scalar_to_fused_array`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`、`len` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `qkv_idxs`、`shard_id` 以及 `loaded_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 132-145: Function `adjust_shard_offsets` and its core logic
```python
def adjust_shard_offsets(shard_offsets, loaded_weight, dim):
    actual_weight_size = loaded_weight.size(dim)
    target_weight_size = shard_offsets[-1][-1] + shard_offsets[-1][-2]
    if actual_weight_size != target_weight_size:
        new_shard_offsets = []
        new_offset = 0
        for shard_id, shard_offset, shard_size in shard_offsets:
            actual_shard_size = actual_weight_size * shard_size // target_weight_size
            new_shard_offsets.append((shard_id, new_offset, actual_shard_size))
            new_offset += actual_shard_size
        return new_shard_offsets
    return shard_offsets
```
**EN:** This block defines `adjust_shard_offsets` and contains the main logic for this step. It mainly invokes `loaded_weight.size` and `new_shard_offsets.append`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `actual_weight_size`, `target_weight_size`, `new_shard_offsets`, `new_offset`, and `actual_shard_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `adjust_shard_offsets`，并承载这一阶段的核心逻辑。 它主要调用 `loaded_weight.size` 和 `new_shard_offsets.append`，说明该流程会编排底层辅助函数或计算内核。 像 `actual_weight_size`、`target_weight_size`、`new_shard_offsets`、`new_offset` 以及 `actual_shard_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 146-157: Class `LinearBase` declaration and shared state
```python
class LinearBase(torch.nn.Module):
    """Base linear layer.

    Args:
        input_size: input dimension of the linear layer.
        output_size: output dimension of the linear layer.
        bias: If true, add bias.
        skip_bias_add: If true, skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
    """
```
**EN:** This block introduces class `LinearBase` and the state shared by its methods. It inherits from `torch.nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Base linear layer.
**CN:** 该代码块引入类 `LinearBase`，并定义其方法共享的状态。 它继承自 `torch.nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 158-190: `LinearBase` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        skip_bias_add: bool = False,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        # Keep input parameters
        self.input_size = input_size
        self.output_size = output_size
        self.skip_bias_add = skip_bias_add
        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        self.params_dtype = params_dtype
        self.quant_config = quant_config
        if quant_config is None:
            from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod

            self.quant_method: Optional[QuantizeMethodBase] = UnquantizedLinearMethod()
        else:
            self.quant_method = quant_config.get_quant_method(self, prefix=prefix)

        if self.quant_method is not None:
            wrap_method_with_debug_kernel_once(
                self.quant_method,
                "apply",
                op_name=f"sglang.quant_method.{self.quant_method.__class__.__name__}.apply",
            )
```
**EN:** This block defines `LinearBase.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `torch.get_default_dtype`, `UnquantizedLinearMethod`, `quant_config.get_quant_method`, and `wrap_method_with_debug_kernel_once`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.input_size`, `self.output_size`, `self.skip_bias_add`, `self.params_dtype`, and `self.quant_config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `LinearBase.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`torch.get_default_dtype`、`UnquantizedLinearMethod`、`quant_config.get_quant_method` 以及 `wrap_method_with_debug_kernel_once`，说明该流程会编排底层辅助函数或计算内核。 像 `self.input_size`、`self.output_size`、`self.skip_bias_add`、`self.params_dtype` 以及 `self.quant_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 191-194: `LinearBase.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This block defines `LinearBase.forward` and contains the main logic for this step.
**CN:** 该代码块定义了 `LinearBase.forward`，并承载这一阶段的核心逻辑。

### Lines 195-208: Class `ReplicatedLinear` declaration and shared state
```python
class ReplicatedLinear(LinearBase):
    """Replicated linear layer.

    Args:
        input_size: input dimension of the linear layer.
        output_size: output dimension of the linear layer.
        bias: If true, add bias.
        skip_bias_add: If true, skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        prefix: The name of the layer in the state dict, including all parents
                        (e.g. model.layers.0.qkv_proj)
    """
```
**EN:** This block introduces class `ReplicatedLinear` and the state shared by its methods. It inherits from `LinearBase`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Replicated linear layer.
**CN:** 该代码块引入类 `ReplicatedLinear`，并定义其方法共享的状态。 它继承自 `LinearBase`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 209-253: `ReplicatedLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        bias: bool = True,
        skip_bias_add: bool = False,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(
            input_size,
            output_size,
            skip_bias_add,
            params_dtype,
            quant_config,
            prefix=prefix,
        )

        # All the linear layer supports quant method.
        assert self.quant_method is not None
        self.quant_method.create_weights(
            self,
            self.input_size,
            [self.output_size],
            self.input_size,
            self.output_size,
            self.params_dtype,
            weight_loader=self.weight_loader,
        )

        if bias:
            self.bias = Parameter(
                torch.empty(self.output_size, dtype=self.params_dtype)
            )
            set_weight_attrs(
                self.bias,
                {
                    "output_dim": 0,
                    "weight_loader": self.weight_loader,
                },
            )
        else:
            self.register_parameter("bias", None)
```
**EN:** This block defines `ReplicatedLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `self.quant_method.create_weights`, `Parameter`, `set_weight_attrs`, and `self.register_parameter`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.bias` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ReplicatedLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`self.quant_method.create_weights`、`Parameter`、`set_weight_attrs` 以及 `self.register_parameter`，说明该流程会编排底层辅助函数或计算内核。 像 `self.bias` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 254-277: Function `ReplicatedLinear.weight_loader` and its core logic
```python
    def weight_loader(self, param: Parameter, loaded_weight: torch.Tensor):
        # If the weight on disk does not have a shape, give it one
        # (such scales for AutoFp8).
        if len(loaded_weight.shape) == 0:
            loaded_weight = loaded_weight.reshape(1)

        # The per-tensor quant-scale must be 1 dimension
        if _is_npu:
            if param.size() != loaded_weight.size() and param.size(0) == 1:
                if torch.allclose(loaded_weight, loaded_weight[0]):
                    loaded_weight = loaded_weight[:1]
                else:
                    raise ValueError(f"{loaded_weight} are not all equal")

            if param.dtype == torch.int8 or loaded_weight.dtype == torch.int8:
                assert (
                    param.dtype == loaded_weight.dtype
                ), "init para dtype and loaded weight dtype should be the same"

        assert (
            param.size() == loaded_weight.size()
        ), f"{param.shape=} {param.dtype=} {loaded_weight.shape=} {loaded_weight.dtype=}"
        param.data.copy_(loaded_weight)
```
**EN:** This block defines `ReplicatedLinear.weight_loader` and contains the main logic for this step. It mainly invokes `param.data.copy_`, `len`, `loaded_weight.reshape`, `param.size`, and `loaded_weight.size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `loaded_weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ReplicatedLinear.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `param.data.copy_`、`len`、`loaded_weight.reshape`、`param.size` 以及 `loaded_weight.size`，说明该流程会编排底层辅助函数或计算内核。 像 `loaded_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 278-284: `ReplicatedLinear.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> Tuple[torch.Tensor, Optional[torch.Tensor]]:
        bias = self.bias if not self.skip_bias_add else None
        assert self.quant_method is not None
        output = self.quant_method.apply(self, x, bias)
        output_bias = self.bias if self.skip_bias_add else None
        return output, output_bias
```
**EN:** This block defines `ReplicatedLinear.forward` and contains the main logic for this step. It mainly invokes `self.quant_method.apply`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bias`, `output`, and `output_bias` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ReplicatedLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.quant_method.apply`，说明该流程会编排底层辅助函数或计算内核。 像 `bias`、`output` 以及 `output_bias` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 285-291: Function `ReplicatedLinear.extra_repr` and its core logic
```python
    def extra_repr(self) -> str:
        s = f"in_features={self.input_size}"
        s += f", output_features={self.output_size}"
        s += f", bias={self.bias is not None}"
        return s
```
**EN:** This block defines `ReplicatedLinear.extra_repr` and contains the main logic for this step. Intermediate names such as `s` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ReplicatedLinear.extra_repr`，并承载这一阶段的核心逻辑。 像 `s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 292-315: Class `ColumnParallelLinear` declaration and shared state
```python
class ColumnParallelLinear(LinearBase):
    """Linear layer with column parallelism.

    The linear layer is defined as Y = XA + b. A is parallelized along
    its second dimension as A = [A_1, ..., A_p].

    Args:
        input_size: first dimension of matrix A.
        output_size: second dimension of matrix A.
        bias: If true, add bias.
        gather_output: If true, call all-gather on output and make Y available
                       to all GPUs, otherwise, every GPU will have its output
                       which is Y_i = XA_i
        skip_bias_add: This was added to enable performance optimizations where
                       bias can be fused with other element-wise operations. we
                       skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        output_sizes: list of output sizes packed into one output, like for QKV
                       the list would be size 3.
        prefix: The name of the layer in the state dict, including all parents
                        (e.g. model.layers.0.qkv_proj)
    """
```
**EN:** This block introduces class `ColumnParallelLinear` and the state shared by its methods. It inherits from `LinearBase`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Linear layer with column parallelism.
**CN:** 该代码块引入类 `ColumnParallelLinear`，并定义其方法共享的状态。 它继承自 `LinearBase`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 316-384: `ColumnParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        bias: bool = True,
        gather_output: bool = False,
        skip_bias_add: bool = False,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        output_sizes: Optional[List[int]] = None,
        prefix: str = "",
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
        use_presharded_weights: bool = False,
        skip_block_quant_check: bool = False,
    ):
        super().__init__(
            input_size, output_size, skip_bias_add, params_dtype, quant_config, prefix
        )

        self.gather_output = gather_output
        self.use_presharded_weights = use_presharded_weights

        # Divide the weight matrix along the last dimension.
        if tp_rank is None:
            tp_rank = get_tensor_model_parallel_rank()
        if tp_size is None:
            tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank, self.tp_size = tp_rank, tp_size
        assert self.quant_method is not None
        self.output_size_per_partition = divide(self.output_size, tp_size)
        self.output_partition_sizes = [self.output_size_per_partition]
        # If QKV or MergedColumn, use output size of each partition.
        if hasattr(self, "output_sizes"):
            self.output_partition_sizes = [
                divide(output_size, tp_size) for output_size in self.output_sizes
            ]

        if output_sizes is None:
            output_sizes = [output_size]

        self.quant_method.create_weights(
            layer=self,
            input_size_per_partition=self.input_size,
            output_partition_sizes=self.output_partition_sizes,
            input_size=self.input_size,
            output_size=self.output_size,
            params_dtype=self.params_dtype,
            skip_block_quant_check=skip_block_quant_check,
            weight_loader=(
                self.weight_loader_v2
                if self.quant_method.__class__.__name__ in WEIGHT_LOADER_V2_SUPPORTED
                else self.weight_loader
            ),
        )
        if bias:
            self.bias = Parameter(
                torch.zeros(self.output_size_per_partition, dtype=params_dtype)
            )
            set_weight_attrs(
                self.bias,
                {
                    "output_dim": 0,
                    "weight_loader": self.weight_loader,
                },
            )
        else:
            self.register_parameter("bias", None)
```
**EN:** This block defines `ColumnParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `divide`, `hasattr`, `self.quant_method.create_weights`, and `get_tensor_model_parallel_rank`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.gather_output`, `self.use_presharded_weights`, `self.tp_rank`, `self.tp_size`, and `self.output_size_per_partition` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ColumnParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`divide`、`hasattr`、`self.quant_method.create_weights` 以及 `get_tensor_model_parallel_rank`，说明该流程会编排底层辅助函数或计算内核。 像 `self.gather_output`、`self.use_presharded_weights`、`self.tp_rank`、`self.tp_size` 以及 `self.output_size_per_partition` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 385-439: Function `ColumnParallelLinear.weight_loader` and its core logic
```python
    def weight_loader(self, param: Parameter, loaded_weight: torch.Tensor):
        output_dim = getattr(param, "output_dim", None)
        param_data = param.data

        # Special case for GGUF
        is_gguf_weight = getattr(param, "is_gguf_weight", False)
        is_gguf_weight_type = getattr(param, "is_gguf_weight_type", False)
        if is_gguf_weight_type:
            param.weight_type = loaded_weight.item()

        # Materialize GGUF UninitializedParameter
        if is_gguf_weight and isinstance(param, UninitializedParameter):
            weight_shape = list(loaded_weight.shape)
            if output_dim is not None:
                weight_shape[output_dim] = weight_shape[output_dim] // self.tp_size
            param.materialize(tuple(weight_shape), dtype=loaded_weight.dtype)
            param_data = param.data

        # bitsandbytes loads the weights of the specific portion
        # no need to narrow here
        use_bitsandbytes_4bit = getattr(param, "use_bitsandbytes_4bit", False)
        if output_dim is not None and not use_bitsandbytes_4bit:
            shard_size = param_data.shape[output_dim]
            start_idx = self.tp_rank * shard_size

            if _is_cpu:
                from sglang.srt.model_loader.weight_utils import (
                    narrow_padded_param_and_loaded_weight,
                )

                param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                    param_data,
                    loaded_weight,
                    0,  # param_data_start
                    start_idx,
                    output_dim,
                    shard_size,
                    not self.use_presharded_weights,
                )
            else:
                if not self.use_presharded_weights:
                    loaded_weight = loaded_weight.narrow(
                        output_dim, start_idx, shard_size
                    )

        # Special case for loading scales off disk, which often do not
        # have a shape (such as in the case of AutoFP8).
        if len(loaded_weight.shape) == 0:
            loaded_weight = loaded_weight.reshape(1)

        assert (
            param_data.shape == loaded_weight.shape
        ), f"param_data.shape={param_data.shape} != loaded_weight.shape={loaded_weight.shape}"
        param_data.copy_(loaded_weight)
```
**EN:** This block defines `ColumnParallelLinear.weight_loader` and contains the main logic for this step. It mainly invokes `getattr`, `param_data.copy_`, `loaded_weight.item`, `isinstance`, and `list`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output_dim`, `param_data`, `is_gguf_weight`, `is_gguf_weight_type`, and `use_bitsandbytes_4bit` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ColumnParallelLinear.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`、`param_data.copy_`、`loaded_weight.item`、`isinstance` 以及 `list`，说明该流程会编排底层辅助函数或计算内核。 像 `output_dim`、`param_data`、`is_gguf_weight`、`is_gguf_weight_type` 以及 `use_bitsandbytes_4bit` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 440-466: Function `ColumnParallelLinear.weight_loader_v2` and its core logic
```python
    def weight_loader_v2(self, param: Parameter, loaded_weight: torch.Tensor):
        # Special case for loading scales off disk, which often do not
        # have a shape (such as in the case of AutoFP8).
        if len(loaded_weight.shape) == 0:
            assert loaded_weight.numel() == 1
            loaded_weight = loaded_weight.reshape(1)

        if isinstance(param, _ColumnvLLMParameter):
            param.load_column_parallel_weight(
                loaded_weight,
                tp_rank=self.tp_rank,
                use_presharded_weights=self.use_presharded_weights,
            )
        else:
            # FIXME: This branch is needed to load deepseek v3 awq.
            # However, we should fix this and avoid the branching here.
            # After QuantizedRL reload, params might still need tp_rank
            try:
                param.load_column_parallel_weight(
                    loaded_weight,
                    tp_rank=self.tp_rank,
                    use_presharded_weights=self.use_presharded_weights,
                )
            except TypeError:
                # Fallback for parameters that don't accept additional args
                param.load_column_parallel_weight(loaded_weight)
```
**EN:** This block defines `ColumnParallelLinear.weight_loader_v2` and contains the main logic for this step. It mainly invokes `isinstance`, `len`, `loaded_weight.reshape`, `param.load_column_parallel_weight`, and `loaded_weight.numel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `loaded_weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ColumnParallelLinear.weight_loader_v2`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`、`len`、`loaded_weight.reshape`、`param.load_column_parallel_weight` 以及 `loaded_weight.numel`，说明该流程会编排底层辅助函数或计算内核。 像 `loaded_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 467-480: `ColumnParallelLinear.forward` main forward path
```python
    def forward(self, input_):
        bias = self.bias if not self.skip_bias_add else None

        # Matrix multiply.
        assert self.quant_method is not None
        output_parallel = self.quant_method.apply(self, input_, bias)
        if self.gather_output:
            # All-gather across the partitions.
            output = tensor_model_parallel_all_gather(output_parallel)
        else:
            output = output_parallel
        output_bias = self.bias if self.skip_bias_add else None
        return output, output_bias
```
**EN:** This block defines `ColumnParallelLinear.forward` and contains the main logic for this step. It mainly invokes `self.quant_method.apply` and `tensor_model_parallel_all_gather`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bias`, `output_parallel`, `output_bias`, and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ColumnParallelLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.quant_method.apply` 和 `tensor_model_parallel_all_gather`，说明该流程会编排底层辅助函数或计算内核。 像 `bias`、`output_parallel`、`output_bias` 以及 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 481-489: Function `ColumnParallelLinear.extra_repr` and its core logic
```python
    def extra_repr(self) -> str:
        s = f"in_features={self.input_size}"
        s += f", output_features={self.output_size_per_partition}"
        s += f", bias={self.bias is not None}"
        s += f", tp_size={self.tp_size}"
        s += f", gather_output={self.gather_output}"
        return s
```
**EN:** This block defines `ColumnParallelLinear.extra_repr` and contains the main logic for this step. Intermediate names such as `s` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ColumnParallelLinear.extra_repr`，并承载这一阶段的核心逻辑。 像 `s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 490-512: Class `MergedColumnParallelLinear` declaration and shared state
```python
class MergedColumnParallelLinear(ColumnParallelLinear):
    """Packed linear layers with column parallelism.

    Similar to ColumnParallelLinear, but the weight matrix is concatenated
    along the output dimension. When the weight matrix is loaded, the
    different partitions are sharded separately.

    Args:
        input_size: input dimension of the linear layer.
        output_sizes: list of output dimensions of the linear layer.
        bias: If true, add bias.
        gather_output: If true, call all-gather on output and make the output
                       available to all GPUs, otherwise, every GPU will have
                       its own output.
        skip_bias_add: This was added to enable performance optimizations where
                       bias can be fused with other element-wise operations. we
                       skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        prefix: The name of the layer in the state dict, including all parents
                        (e.g. model.layers.0.qkv_proj)
    """
```
**EN:** This block introduces class `MergedColumnParallelLinear` and the state shared by its methods. It inherits from `ColumnParallelLinear`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Packed linear layers with column parallelism.
**CN:** 该代码块引入类 `MergedColumnParallelLinear`，并定义其方法共享的状态。 它继承自 `ColumnParallelLinear`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 513-549: `MergedColumnParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        output_sizes: List[int],
        bias: bool = True,
        gather_output: bool = False,
        skip_bias_add: bool = False,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
        use_presharded_weights: bool = False,
    ):
        self.output_sizes = output_sizes
        if tp_rank is None:
            tp_rank = get_tensor_model_parallel_rank()
        if tp_size is None:
            tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank, self.tp_size = tp_rank, tp_size
        assert all(output_size % tp_size == 0 for output_size in output_sizes)
        self.use_presharded_weights = use_presharded_weights
        super().__init__(
            input_size=input_size,
            output_size=sum(output_sizes),
            bias=bias,
            gather_output=gather_output,
            skip_bias_add=skip_bias_add,
            params_dtype=params_dtype,
            quant_config=quant_config,
            prefix=prefix,
            tp_rank=tp_rank,
            tp_size=tp_size,
            use_presharded_weights=use_presharded_weights,
        )
        self.prefix = prefix
```
**EN:** This block defines `MergedColumnParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `all`, `super.__init__`, `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`, and `sum`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.output_sizes`, `self.tp_rank`, `self.tp_size`, `self.use_presharded_weights`, and `self.prefix` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MergedColumnParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `all`、`super.__init__`、`get_tensor_model_parallel_rank`、`get_tensor_model_parallel_world_size` 以及 `sum`，说明该流程会编排底层辅助函数或计算内核。 像 `self.output_sizes`、`self.tp_rank`、`self.tp_size`、`self.use_presharded_weights` 以及 `self.prefix` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 550-727: Function `MergedColumnParallelLinear.weight_loader` and its core logic
```python
    def weight_loader(
        self,
        param: Parameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: tuple[int, ...] | int | None = None,
    ):
        if isinstance(loaded_shard_id, tuple):
            if hasattr(param, "load_merged_column_weight"):
                return self.weight_loader_v2(param, loaded_weight, loaded_shard_id)
            raise NotImplementedError(
                "Shard id with multiple indices is not supported in weight_loader, "
                "please use weight_loader_v2 instead."
            )

        # Special case for GGUF
        # initialize GGUF param after we know the quantize type
        is_gguf_weight = getattr(param, "is_gguf_weight", False)
        is_gguf_weight_type = getattr(param, "is_gguf_weight_type", False)
        if is_gguf_weight_type:
            param.data[loaded_shard_id].copy_(loaded_weight)
            param.shard_weight_type[loaded_shard_id] = loaded_weight.item()
            return

        if is_gguf_weight:
            output_dim = getattr(param, "output_dim", None)
            shard_size = loaded_weight.size(output_dim) // self.tp_size
            start_idx = self.tp_rank * shard_size

            loaded_weight = loaded_weight.narrow(output_dim, start_idx, shard_size)

            param.shard_id.append(loaded_shard_id)
            param.shard_id_map[loaded_shard_id] = len(param.data_container)
            param.data_container.append(loaded_weight)
            return

        param_data = param.data
        output_dim = getattr(param, "output_dim", None)
        # Special case for AQLM codebooks.
        is_metadata = getattr(param, "is_metadata", False)
        # Special case for per-tensor scale to load scalar into fused array.
        needs_scalar_to_array = getattr(param, "needs_scalar_to_array", False)

        if loaded_shard_id is None:
            # Loaded weight is already fused on disk (qkv/mlp).
            if output_dim is None:
                if needs_scalar_to_array:
                    param_data, loaded_weight = adjust_scalar_to_fused_array(
                        param_data, loaded_weight, 0
                    )

                assert param_data.shape == loaded_weight.shape
                param_data.copy_(loaded_weight)
                return
            current_shard_offset = 0
            shard_offsets: List[Tuple[int, int, int]] = []
            for i, output_size in enumerate(self.output_sizes):
                effective_size = (
                    output_size // self.tp_size
                    if self.use_presharded_weights
                    else output_size
                )
                shard_offsets.append((i, current_shard_offset, effective_size))
                current_shard_offset += effective_size
            packed_dim = getattr(param, "packed_dim", None)

            use_bitsandbytes_4bit = getattr(param, "use_bitsandbytes_4bit", False)
            if _is_cpu:
                shard_offsets = adjust_shard_offsets(
                    shard_offsets, loaded_weight, output_dim
                )

            for shard_id, shard_offset, shard_size in shard_offsets:
                # Special case for Quantization.
                # If quantized, we need to adjust the offset and size to account
                # for the packing.
                if packed_dim == output_dim:
                    shard_size = shard_size // param.pack_factor
                    shard_offset = shard_offset // param.pack_factor
                    # Special case for Marlin.
                    shard_size, shard_offset = adjust_marlin_shard(
                        param, shard_size, shard_offset
                    )

                if use_bitsandbytes_4bit:
                    index = list(itertools.accumulate([0] + self.output_sizes))
                    orig_offsets = {
                        str(i): (index[i], size)
                        for i, size in enumerate(self.output_sizes)
                    }
                    orig_offsets["total"] = (self.output_size, 0)
                    shard_size, shard_offset = adjust_bitsandbytes_4bit_shard(
                        param, orig_offsets, str(shard_id)
                    )

                loaded_weight_shard = loaded_weight.narrow(
                    output_dim, shard_offset, shard_size
                )
                self.weight_loader(param, loaded_weight_shard, shard_id)
            return

        assert loaded_shard_id < len(self.output_sizes)
        if output_dim is not None:
            shard_offset = sum(self.output_sizes[:loaded_shard_id]) // self.tp_size
            shard_size = self.output_sizes[loaded_shard_id] // self.tp_size
            # Special case for quantization.
            # If quantized, we need to adjust the offset and size to account
            # for the packing.
            packed_dim = getattr(param, "packed_dim", None)
            if packed_dim == output_dim:
                shard_size = shard_size // param.pack_factor
                shard_offset = shard_offset // param.pack_factor
                # Special case for Marlin.
                shard_size, shard_offset = adjust_marlin_shard(
                    param, shard_size, shard_offset
                )

            use_bitsandbytes_4bit = getattr(param, "use_bitsandbytes_4bit", False)
            if use_bitsandbytes_4bit:
                shard_size = loaded_weight.shape[output_dim]
                shard_offset = loaded_weight.shape[output_dim] * loaded_shard_id

            param_data = param_data.narrow(output_dim, shard_offset, shard_size)
            start_idx = self.tp_rank * shard_size

            if _is_cpu:
                from sglang.srt.model_loader.weight_utils import (
                    narrow_padded_param_and_loaded_weight,
                )

                param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                    param_data,
                    loaded_weight,
                    0,  # param_data_start
                    start_idx,
                    output_dim,
                    shard_size,
                    not use_bitsandbytes_4bit and not self.use_presharded_weights,
                )
            else:
                # bitsandbytes loads the weights of the specific portion
                # no need to narrow here
                if not use_bitsandbytes_4bit and not self.use_presharded_weights:
                    # Padding for special case like qwen2_5_VL's mlp which is not 8-aligned
                    end_idx = start_idx + shard_size
                    if end_idx > loaded_weight.shape[output_dim]:
                        loaded_weight = pad_or_narrow_weight(
                            loaded_weight, output_dim, start_idx, shard_size
                        )
                    else:
                        loaded_weight = loaded_weight.narrow(
                            output_dim, start_idx, shard_size
                        )

        # Special case for AQLM codebooks.
        elif is_metadata:
            # metadata indicates fixed size concatenated along dim 0
            shard_size = loaded_weight.shape[0]
            shard_offset = loaded_shard_id * shard_size
            param_data = param_data.narrow(0, shard_offset, shard_size)

        # Special case for per-tensor scales in fused case.
        elif needs_scalar_to_array:
            param_data, loaded_weight = adjust_scalar_to_fused_array(
                param_data, loaded_weight, loaded_shard_id
            )

        else:
            ignore_warning = getattr(param, "ignore_warning", False)
            if not ignore_warning:
                logger.warning(
                    "Loading a weight without `output_dim` attribute in "
                    "MergedColumnParallelLinear, assume the weight is "
                    "the same for all partitions."
                )

        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines `MergedColumnParallelLinear.weight_loader` and contains the main logic for this step. It mainly invokes `isinstance`, `getattr`, `param_data.copy_`, `hasattr`, and `NotImplementedError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_gguf_weight`, `is_gguf_weight_type`, `param_data`, `output_dim`, and `is_metadata` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MergedColumnParallelLinear.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`、`getattr`、`param_data.copy_`、`hasattr` 以及 `NotImplementedError`，说明该流程会编排底层辅助函数或计算内核。 像 `is_gguf_weight`、`is_gguf_weight_type`、`param_data`、`output_dim` 以及 `is_metadata` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 728-774: Internal helper `MergedColumnParallelLinear._load_fused_module_from_checkpoint`
```python
    def _load_fused_module_from_checkpoint(
        self,
        param: BasevLLMParameter,
        loaded_weight: torch.Tensor,
        output_sizes: list[int] | None = None,
    ):
        """
        Handle special case for models where MLP layers are already
        fused on disk. In this case, we have no shard id. This function
        determmines the shard id by splitting these layers and then calls
        the weight loader using the shard id.

        An example of a model with these fused layers:
        https://huggingface.co/microsoft/Phi-3-mini-4k-instruct
        """

        current_shard_offset = 0
        shard_offsets: List[Tuple[int, int, int]] = []
        output_sizes = output_sizes or self.output_sizes
        for i, output_size in enumerate(output_sizes):
            shard_offsets.append((i, current_shard_offset, output_size))
            current_shard_offset += output_size
        if _is_cpu:
            from sglang.srt.model_loader.weight_utils import (
                pad_loaded_weight,
            )

            loaded_weight = pad_loaded_weight(
                loaded_weight, param.output_dim, output_sizes
            )

        for shard_id, shard_offset, shard_size in shard_offsets:
            # Special case for Quantization.
            # If quantized, we need to adjust the offset and size to account
            # for the packing.
            if (
                isinstance(param, (PackedColumnParameter, PackedvLLMParameter))
                and param.packed_dim == param.output_dim
            ):
                shard_size, shard_offset = param.adjust_shard_indexes_for_packing(
                    shard_size=shard_size, shard_offset=shard_offset
                )
            loaded_weight_shard = loaded_weight.narrow(
                param.output_dim, shard_offset, shard_size
            )
            self.weight_loader_v2(param, loaded_weight_shard, shard_id)
```
**EN:** This block defines `MergedColumnParallelLinear._load_fused_module_from_checkpoint` and contains the main logic for this step. It mainly invokes `enumerate`, `shard_offsets.append`, `pad_loaded_weight`, `loaded_weight.narrow`, and `self.weight_loader_v2`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `current_shard_offset`, `shard_offsets`, `output_sizes`, `loaded_weight`, and `loaded_weight_shard` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MergedColumnParallelLinear._load_fused_module_from_checkpoint`，并承载这一阶段的核心逻辑。 它主要调用 `enumerate`、`shard_offsets.append`、`pad_loaded_weight`、`loaded_weight.narrow` 以及 `self.weight_loader_v2`，说明该流程会编排底层辅助函数或计算内核。 像 `current_shard_offset`、`shard_offsets`、`output_sizes`、`loaded_weight` 以及 `loaded_weight_shard` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 775-828: Internal helper `MergedColumnParallelLinear._load_merged_block_scale`
```python
    def _load_merged_block_scale(
        self, param: BasevLLMParameter, loaded_weight: torch.Tensor
    ):
        """
        Handle block-wise scale loading for MergedColumnParallelLinear.
        Similar to QKVParallelLinear._load_qkv_block_scale, but for merged column layers.
        """
        weight_block_size = self.quant_method.quant_config.weight_block_size
        block_n, _ = weight_block_size[0], weight_block_size[1]
        block_n = 1 if getattr(param, "format_ue8m0", False) else block_n

        # Calculate block sizes for each shard
        shard_block_sizes = []
        shard_block_offsets = []
        current_block_offset = 0
        for output_size in self.output_sizes:
            shard_block_size = (output_size + block_n - 1) // block_n
            shard_block_sizes.append(shard_block_size)
            shard_block_offsets.append(current_block_offset)
            current_block_offset += shard_block_size

        if _is_cpu:
            from sglang.srt.model_loader.weight_utils import (
                pad_loaded_weight,
            )

            loaded_weight = pad_loaded_weight(
                loaded_weight, param.output_dim, shard_block_sizes
            )

        # Load each shard
        for shard_id, (shard_block_offset, shard_block_size) in enumerate(
            zip(shard_block_offsets, shard_block_sizes)
        ):
            # Extract the shard from loaded_weight
            loaded_weight_shard = loaded_weight.narrow(
                param.output_dim, shard_block_offset, shard_block_size
            )

            # Calculate per-rank offset and size (considering TP)
            rank_shard_offset = shard_block_offset // self.tp_size
            rank_shard_size = shard_block_size // self.tp_size

            # Load into the parameter
            param.load_merged_column_weight(
                loaded_weight=loaded_weight_shard,
                shard_id=shard_id,
                shard_offset=rank_shard_offset,
                shard_size=rank_shard_size,
                tp_rank=self.tp_rank,
                tp_size=self.tp_size,
                use_presharded_weights=self.use_presharded_weights,
            )
```
**EN:** This block defines `MergedColumnParallelLinear._load_merged_block_scale` and contains the main logic for this step. It mainly invokes `enumerate`, `getattr`, `shard_block_sizes.append`, `shard_block_offsets.append`, and `pad_loaded_weight`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `weight_block_size`, `block_n`, `_`, `shard_block_sizes`, and `shard_block_offsets` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MergedColumnParallelLinear._load_merged_block_scale`，并承载这一阶段的核心逻辑。 它主要调用 `enumerate`、`getattr`、`shard_block_sizes.append`、`shard_block_offsets.append` 以及 `pad_loaded_weight`，说明该流程会编排底层辅助函数或计算内核。 像 `weight_block_size`、`block_n`、`_`、`shard_block_sizes` 以及 `shard_block_offsets` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 829-893: Function `MergedColumnParallelLinear.weight_loader_v2` and its core logic
```python
    def weight_loader_v2(
        self,
        param: BasevLLMParameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: tuple[int, ...] | int | None = None,
    ):
        if loaded_shard_id is None or isinstance(loaded_shard_id, tuple):
            if isinstance(param, PerTensorScaleParameter):
                param.load_merged_column_weight(
                    loaded_weight=loaded_weight,
                    shard_id=0,
                    tp_rank=self.tp_rank,
                    tp_size=self.tp_size,
                )
                return
            elif isinstance(param, BlockQuantScaleParameter):
                self._load_merged_block_scale(param, loaded_weight)
                return
            elif type(param) in (RowvLLMParameter, BasevLLMParameter):
                param.load_merged_column_weight(
                    loaded_weight=loaded_weight,
                    tp_rank=self.tp_rank,
                    tp_size=self.tp_size,
                )
                return
            output_sizes = (
                [self.output_sizes[idx] for idx in loaded_shard_id]
                if loaded_shard_id
                else None
            )
            # TODO: @dsikka - move to parameter.py
            self._load_fused_module_from_checkpoint(
                param, loaded_weight, output_sizes=output_sizes
            )
            return

        assert loaded_shard_id < len(self.output_sizes)

        if isinstance(param, BlockQuantScaleParameter):
            weight_block_size = self.quant_method.quant_config.weight_block_size
            raw_block_n, _ = weight_block_size[0], weight_block_size[1]
            block_n = 1 if getattr(param, "format_ue8m0", False) else raw_block_n
            shard_offset = (
                (sum(self.output_sizes[:loaded_shard_id]) + block_n - 1) // block_n
            ) // self.tp_size
            shard_size = (
                (self.output_sizes[loaded_shard_id] + block_n - 1)
                // block_n
                // self.tp_size
            )
        else:
            shard_offset = sum(self.output_sizes[:loaded_shard_id]) // self.tp_size
            shard_size = self.output_sizes[loaded_shard_id] // self.tp_size

        param.load_merged_column_weight(
            loaded_weight=loaded_weight,
            shard_id=loaded_shard_id,
            shard_offset=shard_offset,
            shard_size=shard_size,
            use_presharded_weights=self.use_presharded_weights,
            tp_rank=self.tp_rank,
            tp_size=self.tp_size,
        )
```
**EN:** This block defines `MergedColumnParallelLinear.weight_loader_v2` and contains the main logic for this step. It mainly invokes `isinstance`, `param.load_merged_column_weight`, `self._load_fused_module_from_checkpoint`, `len`, and `getattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output_sizes`, `weight_block_size`, `raw_block_n`, `_`, and `block_n` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MergedColumnParallelLinear.weight_loader_v2`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`、`param.load_merged_column_weight`、`self._load_fused_module_from_checkpoint`、`len` 以及 `getattr`，说明该流程会编排底层辅助函数或计算内核。 像 `output_sizes`、`weight_block_size`、`raw_block_n`、`_` 以及 `block_n` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 894-919: Class `QKVParallelLinear` declaration and shared state
```python
class QKVParallelLinear(ColumnParallelLinear):
    """Linear layers for the attention's QKV transformation.

    Linear layers for the linear transformation of the query, key, and value
    vectors in the attention layer. The weight matrix is concatenated along
    the output dimension. The layer is parallelized along the head dimension.
    When the number of key/value heads is smaller than the number of query
    heads (e.g., multi-query/grouped-query attention), the key/value head may
    be replicated while the query heads are partitioned.

    Args:
        hidden_size: input hidden state size of the transformer.
        head_size: size of each attention head.
        total_num_heads: total number of attention query heads.
        total_num_kv_heads: total number of attention key/value heads. If
                            None, assume total_num_kv_heads = total_num_heads.
        bias: If true, add bias.
        skip_bias_add: This was added to enable performance optimizations where
                       bias can be fused with other element-wise operations. we
                       skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        prefix: The name of the layer in the state dict, including all parents
                        (e.g. model.layers.0.qkv_proj)
    """
```
**EN:** This block introduces class `QKVParallelLinear` and the state shared by its methods. It inherits from `ColumnParallelLinear`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Linear layers for the attention's QKV transformation.
**CN:** 该代码块引入类 `QKVParallelLinear`，并定义其方法共享的状态。 它继承自 `ColumnParallelLinear`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 920-988: `QKVParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        hidden_size: int,
        head_size: int,
        total_num_heads: int,
        total_num_kv_heads: Optional[int] = None,
        bias: bool = True,
        skip_bias_add: bool = False,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
        load_presharded_attn: bool = False,
        v_head_size: Optional[int] = None,
        skip_block_quant_check: bool = False,
    ):
        self.hidden_size = hidden_size
        self.head_size = head_size
        self.v_head_size = v_head_size if v_head_size is not None else head_size
        self.total_num_heads = total_num_heads
        if total_num_kv_heads is None:
            total_num_kv_heads = total_num_heads
        self.total_num_kv_heads = total_num_kv_heads
        # Divide the weight matrix along the last dimension.
        if tp_rank is None:
            tp_rank = get_tensor_model_parallel_rank()
        if tp_size is None:
            tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank, self.tp_size = tp_rank, tp_size
        self.num_heads = divide(self.total_num_heads, tp_size)
        if tp_size >= self.total_num_kv_heads:
            self.num_kv_heads = 1
            self.num_kv_head_replicas = divide(tp_size, self.total_num_kv_heads)
        else:
            self.num_kv_heads = divide(self.total_num_kv_heads, tp_size)
            self.num_kv_head_replicas = 1
        self.q_proj_shard_size = self.num_heads * self.head_size
        self.kv_proj_shard_size = self.num_kv_heads * self.head_size
        self.v_proj_shard_size = self.num_kv_heads * self.v_head_size
        input_size = self.hidden_size
        output_size = (
            self.num_heads * self.head_size
            + self.num_kv_heads * self.head_size
            + self.num_kv_heads * self.v_head_size
        ) * tp_size
        self.output_sizes = [
            self.num_heads * self.head_size * tp_size,  # q_proj
            self.num_kv_heads * self.head_size * tp_size,  # k_proj
            self.num_kv_heads * self.v_head_size * tp_size,  # v_proj
        ]
        self.use_presharded_weights = load_presharded_attn
        quant_config = None if _disable_hip_linear_quant else quant_config

        super().__init__(
            input_size=input_size,
            output_size=output_size,
            bias=bias,
            gather_output=False,
            skip_bias_add=skip_bias_add,
            params_dtype=params_dtype,
            quant_config=quant_config,
            prefix=prefix,
            tp_rank=tp_rank,
            tp_size=tp_size,
            use_presharded_weights=self.use_presharded_weights,
            skip_block_quant_check=skip_block_quant_check,
        )
```
**EN:** This block defines `QKVParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `divide`, `super.__init__`, `get_tensor_model_parallel_rank`, and `get_tensor_model_parallel_world_size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.hidden_size`, `self.head_size`, `self.v_head_size`, `self.total_num_heads`, and `self.total_num_kv_heads` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `QKVParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `divide`、`super.__init__`、`get_tensor_model_parallel_rank` 以及 `get_tensor_model_parallel_world_size`，说明该流程会编排底层辅助函数或计算内核。 像 `self.hidden_size`、`self.head_size`、`self.v_head_size`、`self.total_num_heads` 以及 `self.total_num_kv_heads` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 989-998: Internal helper `QKVParallelLinear._get_shard_offset_mapping`
```python
    def _get_shard_offset_mapping(self, loaded_shard_id: str):
        shard_offset_mapping = {
            "q": 0,
            "k": self.num_heads * self.head_size,
            "v": (self.num_heads + self.num_kv_heads) * self.head_size,
            "total": (self.num_heads + self.num_kv_heads) * self.head_size
            + self.num_kv_heads * self.v_head_size,
        }
        return shard_offset_mapping.get(loaded_shard_id)
```
**EN:** This block defines `QKVParallelLinear._get_shard_offset_mapping` and contains the main logic for this step. It mainly invokes `shard_offset_mapping.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_offset_mapping` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `QKVParallelLinear._get_shard_offset_mapping`，并承载这一阶段的核心逻辑。 它主要调用 `shard_offset_mapping.get`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_offset_mapping` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 999-1006: Internal helper `QKVParallelLinear._get_shard_size_mapping`
```python
    def _get_shard_size_mapping(self, loaded_shard_id: str):
        shard_size_mapping = {
            "q": self.num_heads * self.head_size,
            "k": self.num_kv_heads * self.head_size,
            "v": self.num_kv_heads * self.v_head_size,
        }
        return shard_size_mapping.get(loaded_shard_id)
```
**EN:** This block defines `QKVParallelLinear._get_shard_size_mapping` and contains the main logic for this step. It mainly invokes `shard_size_mapping.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_size_mapping` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `QKVParallelLinear._get_shard_size_mapping`，并承载这一阶段的核心逻辑。 它主要调用 `shard_size_mapping.get`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_size_mapping` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1007-1051: Internal helper `QKVParallelLinear._load_fused_module_from_checkpoint`
```python
    def _load_fused_module_from_checkpoint(
        self, param: BasevLLMParameter, loaded_weight: torch.Tensor
    ):
        """
        Handle special case for models where QKV layers are already
        fused on disk. In this case, we have no shard id. This function
        determmines the shard id by splitting these layers and then calls
        the weight loader using the shard id.

        An example of a model with these fused layers:
        https://huggingface.co/microsoft/Phi-3-mini-4k-instruct
        """
        shard_offsets = [
            # (shard_id, shard_offset, shard_size)
            ("q", 0, self.total_num_heads * self.head_size),
            (
                "k",
                self.total_num_heads * self.head_size,
                self.total_num_kv_heads * self.head_size,
            ),
            (
                "v",
                (self.total_num_heads + self.total_num_kv_heads) * self.head_size,
                self.total_num_kv_heads * self.v_head_size,
            ),
        ]

        for shard_id, shard_offset, shard_size in shard_offsets:
            # Special case for Quantization.
            # If quantized, we need to adjust the offset and size to account
            # for the packing.
            if (
                isinstance(param, (PackedColumnParameter, PackedvLLMParameter))
                and param.packed_dim == param.output_dim
            ):
                shard_size, shard_offset = param.adjust_shard_indexes_for_packing(
                    shard_size=shard_size, shard_offset=shard_offset
                )

            if not self.use_presharded_weights:
                loaded_weight_shard = loaded_weight.narrow(
                    param.output_dim, shard_offset, shard_size
                )
            self.weight_loader_v2(param, loaded_weight_shard, shard_id)
```
**EN:** This block defines `QKVParallelLinear._load_fused_module_from_checkpoint` and contains the main logic for this step. It mainly invokes `self.weight_loader_v2`, `isinstance`, `param.adjust_shard_indexes_for_packing`, and `loaded_weight.narrow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_offsets`, `shard_size`, `shard_offset`, and `loaded_weight_shard` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `QKVParallelLinear._load_fused_module_from_checkpoint`，并承载这一阶段的核心逻辑。 它主要调用 `self.weight_loader_v2`、`isinstance`、`param.adjust_shard_indexes_for_packing` 以及 `loaded_weight.narrow`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_offsets`、`shard_size`、`shard_offset` 以及 `loaded_weight_shard` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1052-1080: Internal helper `QKVParallelLinear._load_qkv_block_scale`
```python
    def _load_qkv_block_scale(
        self, param: BasevLLMParameter, loaded_weight: torch.Tensor
    ):
        block_n, _ = self.quant_method.quant_config.weight_block_size
        q_size = self.total_num_heads * self.head_size // block_n
        k_size = self.total_num_kv_heads * self.head_size // block_n
        v_size = self.total_num_kv_heads * self.v_head_size // block_n
        shard_offsets = [
            # (shard_id, shard_offset, shard_size)
            ("q", 0, q_size),
            ("k", q_size, k_size),
            ("v", q_size + k_size, v_size),
        ]
        for shard_id, shard_offset, shard_size in shard_offsets:
            loaded_weight_shard = loaded_weight.narrow(
                param.output_dim, shard_offset, shard_size
            )
            rank_shard_offset = self._get_shard_offset_mapping(shard_id) // block_n
            rank_shard_size = self._get_shard_size_mapping(shard_id) // block_n
            param.load_qkv_weight(
                loaded_weight=loaded_weight_shard,
                num_heads=self.num_kv_head_replicas,
                shard_id=shard_id,
                shard_offset=rank_shard_offset,
                shard_size=rank_shard_size,
                tp_rank=self.tp_rank,
                use_presharded_weights=self.use_presharded_weights,
            )
```
**EN:** This block defines `QKVParallelLinear._load_qkv_block_scale` and contains the main logic for this step. It mainly invokes `loaded_weight.narrow`, `param.load_qkv_weight`, `self._get_shard_offset_mapping`, and `self._get_shard_size_mapping`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `block_n`, `_`, `q_size`, `k_size`, and `v_size` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `QKVParallelLinear._load_qkv_block_scale`，并承载这一阶段的核心逻辑。 它主要调用 `loaded_weight.narrow`、`param.load_qkv_weight`、`self._get_shard_offset_mapping` 以及 `self._get_shard_size_mapping`，说明该流程会编排底层辅助函数或计算内核。 像 `block_n`、`_`、`q_size`、`k_size` 以及 `v_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1081-1122: Function `QKVParallelLinear.weight_loader_v2` and its core logic
```python
    def weight_loader_v2(
        self,
        param: BasevLLMParameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: Optional[str] = None,
    ):
        if loaded_shard_id is None:  # special case for certain models
            if isinstance(param, PerTensorScaleParameter):
                param.load_qkv_weight(loaded_weight=loaded_weight, shard_id=0)
                return
            elif type(param) in (RowvLLMParameter, BasevLLMParameter):
                param.load_qkv_weight(loaded_weight=loaded_weight)
                return
            elif isinstance(param, BlockQuantScaleParameter):
                self._load_qkv_block_scale(param, loaded_weight)
                return
            # TODO: @dsikka - move to parameter.py
            self._load_fused_module_from_checkpoint(param, loaded_weight)
            return

        assert loaded_shard_id in ["q", "k", "v"]

        shard_offset = self._get_shard_offset_mapping(loaded_shard_id)
        shard_size = self._get_shard_size_mapping(loaded_shard_id)

        if isinstance(param, BlockQuantScaleParameter):
            weight_block_size = self.quant_method.quant_config.weight_block_size
            raw_block_n, _ = weight_block_size[0], weight_block_size[1]
            block_n = 1 if getattr(param, "format_ue8m0", False) else raw_block_n
            shard_offset = (shard_offset + block_n - 1) // block_n
            shard_size = (shard_size + block_n - 1) // block_n

        param.load_qkv_weight(
            loaded_weight=loaded_weight,
            num_heads=self.num_kv_head_replicas,
            shard_id=loaded_shard_id,
            shard_offset=shard_offset,
            shard_size=shard_size,
            tp_rank=self.tp_rank,
            use_presharded_weights=self.use_presharded_weights,
        )
```
**EN:** This block defines `QKVParallelLinear.weight_loader_v2` and contains the main logic for this step. It mainly invokes `self._get_shard_offset_mapping`, `self._get_shard_size_mapping`, `isinstance`, `param.load_qkv_weight`, and `self._load_fused_module_from_checkpoint`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_offset`, `shard_size`, `weight_block_size`, `raw_block_n`, and `_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `QKVParallelLinear.weight_loader_v2`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_shard_offset_mapping`、`self._get_shard_size_mapping`、`isinstance`、`param.load_qkv_weight` 以及 `self._load_fused_module_from_checkpoint`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_offset`、`shard_size`、`weight_block_size`、`raw_block_n` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1123-1339: Function `QKVParallelLinear.weight_loader` and its core logic
```python
    def weight_loader(
        self,
        param: Parameter,
        loaded_weight: torch.Tensor,
        loaded_shard_id: Optional[str] = None,
    ):

        # Special case for GGUF
        # initialize GGUF param after we know the quantize type
        is_gguf_weight = getattr(param, "is_gguf_weight", False)
        is_gguf_weight_type = getattr(param, "is_gguf_weight_type", False)
        if is_gguf_weight_type and loaded_shard_id is not None:
            idx_map = {"q": 0, "k": 1, "v": 2}
            param.data[idx_map[loaded_shard_id]].copy_(loaded_weight)
            param.shard_weight_type[loaded_shard_id] = loaded_weight.item()
            return

        if is_gguf_weight:
            output_dim = getattr(param, "output_dim", None)
            shard_size = loaded_weight.size(output_dim) // self.tp_size
            start_idx = self.tp_rank * shard_size

            loaded_weight = loaded_weight.narrow(output_dim, start_idx, shard_size)

            param.shard_id.append(loaded_shard_id)
            param.shard_id_map[loaded_shard_id] = len(param.data_container)
            param.data_container.append(loaded_weight)
            return

        param_data = param.data
        output_dim = getattr(param, "output_dim", None)
        # Special case for AQLM codebooks.
        is_metadata = getattr(param, "is_metadata", False)

        # Special case for per-tensor scales in fused case.
        needs_scalar_to_array = getattr(param, "needs_scalar_to_array", False)

        if loaded_shard_id is None:
            # Loaded weight is already fused on disk (qkv/mlp).
            if output_dim is None:
                if needs_scalar_to_array:
                    param_data, loaded_weight = adjust_scalar_to_fused_array(
                        param_data, loaded_weight, 0
                    )

                assert param_data.shape == loaded_weight.shape
                param_data.copy_(loaded_weight)
                return
            shard_offsets = [
                # (shard_id, shard_offset, shard_size)
                ("q", 0, self.total_num_heads * self.head_size),
                (
                    "k",
                    self.total_num_heads * self.head_size,
                    self.total_num_kv_heads * self.head_size,
                ),
                (
                    "v",
                    (self.total_num_heads + self.total_num_kv_heads) * self.head_size,
                    self.total_num_kv_heads * self.v_head_size,
                ),
            ]
            use_bitsandbytes_4bit = getattr(param, "use_bitsandbytes_4bit", False)

            packed_dim = getattr(param, "packed_dim", None)
            if _is_cpu:
                shard_offsets = adjust_shard_offsets(
                    shard_offsets, loaded_weight, output_dim
                )

            for shard_id, shard_offset, shard_size in shard_offsets:
                # Special case for Quantized Weights.
                # If quantized, we need to adjust the offset and size to account
                # for the packing.
                if packed_dim == output_dim:
                    shard_size = shard_size // param.pack_factor
                    shard_offset = shard_offset // param.pack_factor

                    # Special case for Marlin.
                    shard_size, shard_offset = adjust_marlin_shard(
                        param, shard_size, shard_offset
                    )

                if use_bitsandbytes_4bit:
                    orig_qkv_offsets = {
                        "q": (0, self.total_num_heads * self.head_size),
                        "k": (
                            self.total_num_heads * self.head_size,
                            self.total_num_kv_heads * self.head_size,
                        ),
                        "v": (
                            (self.total_num_heads + self.total_num_kv_heads)
                            * self.head_size,
                            self.total_num_kv_heads * self.v_head_size,
                        ),
                        "total": (
                            (self.total_num_heads + self.total_num_kv_heads)
                            * self.head_size
                            + self.total_num_kv_heads * self.v_head_size,
                            0,
                        ),
                    }

                    shard_size, shard_offset = adjust_bitsandbytes_4bit_shard(
                        param, orig_qkv_offsets, shard_id
                    )

                if not self.use_presharded_weights:
                    loaded_weight_shard = loaded_weight.narrow(
                        output_dim, shard_offset, shard_size
                    )
                self.weight_loader(param, loaded_weight_shard, shard_id)
            return

        assert loaded_shard_id in ["q", "k", "v"]

        # If output dim is defined, use the default loading process.
        if output_dim is not None:
            if loaded_shard_id == "q":
                shard_offset = 0
                shard_size = self.num_heads * self.head_size
            elif loaded_shard_id == "k":
                shard_offset = self.num_heads * self.head_size
                shard_size = self.num_kv_heads * self.head_size
            elif loaded_shard_id == "v":
                shard_offset = (self.num_heads + self.num_kv_heads) * self.head_size
                shard_size = self.num_kv_heads * self.v_head_size
            # Special case for Quantized Weights.
            # If quantized, we need to adjust the offset and size to account
            # for the packing.
            packed_dim = getattr(param, "packed_dim", None)
            if packed_dim == output_dim:
                shard_size = shard_size // param.pack_factor
                shard_offset = shard_offset // param.pack_factor

                # Special case for Marlin.
                shard_size, shard_offset = adjust_marlin_shard(
                    param, shard_size, shard_offset
                )

            use_bitsandbytes_4bit = getattr(param, "use_bitsandbytes_4bit", False)
            if use_bitsandbytes_4bit:
                orig_qkv_offsets = {
                    "q": (0, self.num_heads * self.head_size),
                    "k": (
                        self.num_heads * self.head_size,
                        self.num_kv_heads * self.head_size,
                    ),
                    "v": (
                        (self.num_heads + self.num_kv_heads) * self.head_size,
                        self.num_kv_heads * self.v_head_size,
                    ),
                    "total": (
                        (self.num_heads + self.num_kv_heads) * self.head_size
                        + self.num_kv_heads * self.v_head_size,
                        0,
                    ),
                }
                shard_size, shard_offset = adjust_bitsandbytes_4bit_shard(
                    param, orig_qkv_offsets, loaded_shard_id
                )

            param_data = param_data.narrow(output_dim, shard_offset, shard_size)
            if loaded_shard_id == "q":
                shard_id = self.tp_rank
            else:
                shard_id = self.tp_rank // self.num_kv_head_replicas
            start_idx = shard_id * shard_size

            if _is_cpu:
                from sglang.srt.model_loader.weight_utils import (
                    narrow_padded_param_and_loaded_weight,
                )

                param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                    param_data,
                    loaded_weight,
                    0,  # param_data_start
                    start_idx,
                    output_dim,
                    shard_size,
                    not use_bitsandbytes_4bit and not self.use_presharded_weights,
                )
            else:
                # bitsandbytes loads the weights of the specific portion
                # no need to narrow here
                if not use_bitsandbytes_4bit and not self.use_presharded_weights:
                    loaded_weight = loaded_weight.narrow(
                        output_dim, start_idx, shard_size
                    )

        # Special case for AQLM codebooks.
        elif is_metadata:
            # metadata indicates fixed size concatenated along dim 0
            shard_size = loaded_weight.shape[0]
            shard_index = ["q", "k", "v"].index(loaded_shard_id)
            param_data = param_data.narrow(0, shard_index * shard_size, shard_size)
        # Special case for per-tensor scales in fused case.
        elif needs_scalar_to_array:
            param_data, loaded_weight = adjust_scalar_to_fused_array(
                param_data, loaded_weight, loaded_shard_id
            )
        else:
            ignore_warning = getattr(param, "ignore_warning", False)
            if not ignore_warning:
                logger.warning(
                    "Loading a weight without `output_dim` attribute in "
                    "QKVParallelLinear, assume the weight is the same "
                    "for all partitions."
                )

        assert (
            param_data.shape == loaded_weight.shape
        ), f"{param_data.shape=} {loaded_weight.shape=}"
        param_data.copy_(loaded_weight)
```
**EN:** This block defines `QKVParallelLinear.weight_loader` and contains the main logic for this step. It mainly invokes `getattr`, `param_data.copy_`, `param.data.copy_`, `loaded_weight.item`, and `loaded_weight.narrow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_gguf_weight`, `is_gguf_weight_type`, `param_data`, `output_dim`, and `is_metadata` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `QKVParallelLinear.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`、`param_data.copy_`、`param.data.copy_`、`loaded_weight.item` 以及 `loaded_weight.narrow`，说明该流程会编排底层辅助函数或计算内核。 像 `is_gguf_weight`、`is_gguf_weight_type`、`param_data`、`output_dim` 以及 `is_metadata` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1340-1365: Class `RowParallelLinear` declaration and shared state
```python
class RowParallelLinear(LinearBase):
    """Linear layer with row parallelism.

    The linear layer is defined as Y = XA + b. A is parallelized along
    its first dimension and X along its second dimension as:
               -   -
              | A_1 |
              | .   |
          A = | .   |        X = [X_1, ..., X_p]
              | .   |
              | A_p |
               -   -
    Arguments:
        input_size: first dimension of matrix A.
        output_size: second dimension of matrix A.
        bias: If true, add bias. Note that bias is not parallelized.
        input_is_parallel: If true, we assume that the input is already
                           split across the GPUs and we do not split
                           again.
        skip_bias_add: This was added to enable performance optimization where
                       bias can be fused with other element-wise operations.
                       We skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
    """
```
**EN:** This block introduces class `RowParallelLinear` and the state shared by its methods. It inherits from `LinearBase`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Linear layer with row parallelism.
**CN:** 该代码块引入类 `RowParallelLinear`，并定义其方法共享的状态。 它继承自 `LinearBase`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 1366-1426: `RowParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        bias: bool = True,
        input_is_parallel: bool = True,
        skip_bias_add: bool = False,
        params_dtype: Optional[torch.dtype] = None,
        reduce_results: bool = True,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
        use_presharded_weights: bool = False,
        use_dp_attention_reduce: bool = False,
    ):
        quant_config = None if _disable_hip_linear_quant else quant_config
        super().__init__(
            input_size, output_size, skip_bias_add, params_dtype, quant_config, prefix
        )

        self.input_is_parallel = input_is_parallel
        self.reduce_results = reduce_results
        self.use_dp_attention_reduce = use_dp_attention_reduce

        # Divide the weight matrix along the last dimension.
        if tp_rank is None:
            tp_rank = get_tensor_model_parallel_rank()
        if tp_size is None:
            tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank, self.tp_size = tp_rank, tp_size
        self.input_size_per_partition = divide(input_size, self.tp_size)
        assert self.quant_method is not None
        self.use_presharded_weights = use_presharded_weights

        self.quant_method.create_weights(
            layer=self,
            input_size_per_partition=self.input_size_per_partition,
            output_partition_sizes=[self.output_size],
            input_size=self.input_size,
            output_size=self.output_size,
            params_dtype=self.params_dtype,
            weight_loader=(
                self.weight_loader_v2
                if self.quant_method.__class__.__name__ in WEIGHT_LOADER_V2_SUPPORTED
                else self.weight_loader
            ),
        )

        if bias:
            self.bias = Parameter(torch.zeros(self.output_size, dtype=params_dtype))
            set_weight_attrs(
                self.bias,
                {
                    "output_dim": 0,
                    "weight_loader": self.weight_loader,
                },
            )
        else:
            self.register_parameter("bias", None)
```
**EN:** This block defines `RowParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `divide`, `self.quant_method.create_weights`, `get_tensor_model_parallel_rank`, and `get_tensor_model_parallel_world_size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `quant_config`, `self.input_is_parallel`, `self.reduce_results`, `self.use_dp_attention_reduce`, and `self.tp_rank` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RowParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`divide`、`self.quant_method.create_weights`、`get_tensor_model_parallel_rank` 以及 `get_tensor_model_parallel_world_size`，说明该流程会编排底层辅助函数或计算内核。 像 `quant_config`、`self.input_is_parallel`、`self.reduce_results`、`self.use_dp_attention_reduce` 以及 `self.tp_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1427-1489: Function `RowParallelLinear.weight_loader` and its core logic
```python
    def weight_loader(self, param: Parameter, loaded_weight: torch.Tensor):
        input_dim = getattr(param, "input_dim", None)
        use_bitsandbytes_4bit = getattr(param, "use_bitsandbytes_4bit", False)

        # Special case for GGUF
        is_gguf_weight = getattr(param, "is_gguf_weight", False)
        is_gguf_weight_type = getattr(param, "is_gguf_weight_type", False)
        if is_gguf_weight_type:
            param.weight_type = loaded_weight.item()

        # Materialize GGUF UninitializedParameter
        if is_gguf_weight and isinstance(param, UninitializedParameter):
            weight_shape = list(loaded_weight.shape)
            if input_dim:
                weight_shape[input_dim] = weight_shape[input_dim] // self.tp_size
            param.materialize(tuple(weight_shape), dtype=loaded_weight.dtype)

        param_data = param.data
        # bitsandbytes loads the weights of the specific portion
        # no need to narrow here
        if (
            input_dim is not None
            and not use_bitsandbytes_4bit
            and not self.use_presharded_weights
        ):
            shard_size = param_data.shape[input_dim]
            start_idx = self.tp_rank * shard_size

            if _is_cpu:
                from sglang.srt.model_loader.weight_utils import (
                    narrow_padded_param_and_loaded_weight,
                )

                param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                    param_data,
                    loaded_weight,
                    0,  # param_data_start
                    start_idx,
                    input_dim,
                    shard_size,
                )
            else:
                # Padding for special case like qwen2_5_VL's mlp which is not 8-aligned
                end_idx = start_idx + shard_size
                if end_idx > loaded_weight.shape[input_dim]:
                    loaded_weight = pad_or_narrow_weight(
                        loaded_weight, input_dim, start_idx, shard_size
                    )
                else:
                    loaded_weight = loaded_weight.narrow(
                        input_dim, start_idx, shard_size
                    )

        # Special case for loading scales off disk, which often do not
        # have a shape (such as in the case of AutoFP8).
        if len(loaded_weight.shape) == 0:
            loaded_weight = loaded_weight.reshape(1)

        assert (
            param_data.shape == loaded_weight.shape
        ), f"{param_data.shape=} {loaded_weight.shape=}"
        param_data.copy_(loaded_weight)
```
**EN:** This block defines `RowParallelLinear.weight_loader` and contains the main logic for this step. It mainly invokes `getattr`, `param_data.copy_`, `loaded_weight.item`, `isinstance`, and `list`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `input_dim`, `use_bitsandbytes_4bit`, `is_gguf_weight`, `is_gguf_weight_type`, and `param_data` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RowParallelLinear.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`、`param_data.copy_`、`loaded_weight.item`、`isinstance` 以及 `list`，说明该流程会编排底层辅助函数或计算内核。 像 `input_dim`、`use_bitsandbytes_4bit`、`is_gguf_weight`、`is_gguf_weight_type` 以及 `param_data` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1490-1519: Function `RowParallelLinear.weight_loader_v2` and its core logic
```python
    def weight_loader_v2(self, param: BasevLLMParameter, loaded_weight: torch.Tensor):

        # Special case for loading scales off disk, which often do not
        # have a shape (such as in the case of AutoFP8).
        if len(loaded_weight.shape) == 0:
            assert loaded_weight.numel() == 1
            loaded_weight = loaded_weight.reshape(1)

        if isinstance(param, RowvLLMParameter):
            # This `BasevLLMParameter` is defined in sglang/srt/layers/parameter.py,
            # It supports additional parameters like tp_rank and use_presharded_weights.
            param.load_row_parallel_weight(
                loaded_weight,
                tp_rank=self.tp_rank,
                use_presharded_weights=self.use_presharded_weights,
            )
        else:
            # `params` is defined in `vllm/model_executor/parameter.py`,
            # It does not support additional parameters.
            # However, after QuantizedRL reload, params might still need tp_rank
            try:
                param.load_row_parallel_weight(
                    loaded_weight,
                    tp_rank=self.tp_rank,
                    use_presharded_weights=self.use_presharded_weights,
                )
            except TypeError:
                # Fallback for parameters that don't accept additional args
                param.load_row_parallel_weight(loaded_weight)
```
**EN:** This block defines `RowParallelLinear.weight_loader_v2` and contains the main logic for this step. It mainly invokes `isinstance`, `len`, `loaded_weight.reshape`, `param.load_row_parallel_weight`, and `loaded_weight.numel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `loaded_weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RowParallelLinear.weight_loader_v2`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`、`len`、`loaded_weight.reshape`、`param.load_row_parallel_weight` 以及 `loaded_weight.numel`，说明该流程会编排底层辅助函数或计算内核。 像 `loaded_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1520-1565: `RowParallelLinear.forward` main forward path
```python
    def forward(self, input_, skip_all_reduce=False, forward_batch=None):
        if self.input_is_parallel:
            input_parallel = input_
        else:
            splitted_input = split_tensor_along_last_dim(
                input_, num_partitions=self.tp_size
            )
            input_parallel = splitted_input[self.tp_rank].contiguous()

        # Matrix multiply.
        assert self.quant_method is not None
        # Only fuse bias add into GEMM for rank 0 (this ensures that
        # bias will not get added more than once in TP>1 case)
        bias_ = None if (self.tp_rank > 0 or self.skip_bias_add) else self.bias
        if self.use_dp_attention_reduce:
            symm_ctx = use_symmetric_memory(get_attention_tp_group())
        else:
            symm_ctx = use_symmetric_memory(
                get_tp_group(), disabled=not is_allocation_symmetric()
            )
        with symm_ctx:
            output_parallel = self.quant_method.apply(self, input_parallel, bias=bias_)

        if self.reduce_results and self.tp_size > 1 and not skip_all_reduce:
            if self.use_dp_attention_reduce:
                output = get_attention_tp_group().all_reduce(output_parallel)
            else:
                quantize_communications = (
                    (
                        not forward_batch.forward_mode.is_decode_or_idle()
                        and get_global_server_args().enable_quant_communications
                    )
                    if forward_batch is not None
                    else False
                )
                if quantize_communications:
                    output = tensor_model_parallel_quant_all_reduce(output_parallel)
                else:
                    output = tensor_model_parallel_all_reduce(output_parallel)
        else:
            output = output_parallel

        output_bias = self.bias if self.skip_bias_add else None

        return output, output_bias
```
**EN:** This block defines `RowParallelLinear.forward` and contains the main logic for this step. It mainly invokes `split_tensor_along_last_dim`, `splitted_input.contiguous`, `use_symmetric_memory`, `self.quant_method.apply`, and `get_attention_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bias_`, `output_bias`, `input_parallel`, `splitted_input`, and `symm_ctx` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RowParallelLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `split_tensor_along_last_dim`、`splitted_input.contiguous`、`use_symmetric_memory`、`self.quant_method.apply` 以及 `get_attention_tp_group`，说明该流程会编排底层辅助函数或计算内核。 像 `bias_`、`output_bias`、`input_parallel`、`splitted_input` 以及 `symm_ctx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1566-1574: Function `RowParallelLinear.extra_repr` and its core logic
```python
    def extra_repr(self) -> str:
        s = f"input_features={self.input_size_per_partition}"
        s += f", output_features={self.output_size}"
        s += f", bias={self.bias is not None}"
        s += f", tp_size={self.tp_size}"
        s += f", reduce_results={self.reduce_results}"
        return s
```
**EN:** This block defines `RowParallelLinear.extra_repr` and contains the main logic for this step. Intermediate names such as `s` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RowParallelLinear.extra_repr`，并承载这一阶段的核心逻辑。 像 `s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1575-1587: Class `MergedColumnParallelRepeatedLinear` declaration and shared state
```python
class MergedColumnParallelRepeatedLinear(LinearBase):
    """Merged column parallel linear and repeated linear layer.

    TODO: quantization is not supported yet.
    Args:
        input_size: input dimension of the linear layer.
        column_output_sizes: output dimension of the column linear layers.
        repeated_output_sizes: output dimension of the repeated linear layers.
        skip_bias_add: If true, skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
    """
```
**EN:** This block introduces class `MergedColumnParallelRepeatedLinear` and the state shared by its methods. It inherits from `LinearBase`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Merged column parallel linear and repeated linear layer.
**CN:** 该代码块引入类 `MergedColumnParallelRepeatedLinear`，并定义其方法共享的状态。 它继承自 `LinearBase`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 1588-1626: `MergedColumnParallelRepeatedLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        column_output_sizes: List[int],
        repeated_output_sizes: List[int],
        skip_bias_add: bool = False,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        output_size = sum(column_output_sizes) + sum(repeated_output_sizes)
        super().__init__(
            input_size=input_size,
            output_size=output_size,
            skip_bias_add=skip_bias_add,
            params_dtype=params_dtype,
            quant_config=quant_config,
            prefix=prefix,
        )
        self.num_column_parallel = len(column_output_sizes)
        self.tp_rank = get_tensor_model_parallel_rank()
        self.tp_size = get_tensor_model_parallel_world_size()

        self.output_partition_sizes = [
            divide(x, self.tp_size) for x in column_output_sizes
        ] + repeated_output_sizes
        self.quant_method.create_weights(
            layer=self,
            input_size_per_partition=self.input_size,
            output_partition_sizes=self.output_partition_sizes,
            input_size=self.input_size,
            output_size=self.output_size,
            params_dtype=self.params_dtype,
            skip_block_quant_check=True,
            weight_loader=self.weight_loader,
        )

        self.prefix = prefix
```
**EN:** This block defines `MergedColumnParallelRepeatedLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `len`, `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`, and `self.quant_method.create_weights`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output_size`, `self.num_column_parallel`, `self.tp_rank`, `self.tp_size`, and `self.output_partition_sizes` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MergedColumnParallelRepeatedLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`len`、`get_tensor_model_parallel_rank`、`get_tensor_model_parallel_world_size` 以及 `self.quant_method.create_weights`，说明该流程会编排底层辅助函数或计算内核。 像 `output_size`、`self.num_column_parallel`、`self.tp_rank`、`self.tp_size` 以及 `self.output_partition_sizes` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1627-1629: `MergedColumnParallelRepeatedLinear.forward` main forward path
```python
    def forward(self, input_: torch.Tensor) -> torch.Tensor:
        return self.quant_method.apply(self, input_)
```
**EN:** This block defines `MergedColumnParallelRepeatedLinear.forward` and contains the main logic for this step. It mainly invokes `self.quant_method.apply`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MergedColumnParallelRepeatedLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.quant_method.apply`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 1630-1644: Function `MergedColumnParallelRepeatedLinear.weight_loader` and its core logic
```python
    def weight_loader(
        self, param: Parameter, loaded_weight: torch.Tensor, loaded_shard_id: int
    ) -> torch.Tensor:
        output_dim = param.output_dim
        shard_offset = sum(self.output_partition_sizes[:loaded_shard_id])
        shard_size = self.output_partition_sizes[loaded_shard_id]
        param_data = param.data.narrow(output_dim, shard_offset, shard_size)

        if loaded_shard_id < self.num_column_parallel:
            start_idx = self.tp_rank * shard_size
            loaded_weight = loaded_weight.narrow(output_dim, start_idx, shard_size)

        param_data.copy_(loaded_weight)
```
**EN:** This block defines `MergedColumnParallelRepeatedLinear.weight_loader` and contains the main logic for this step. It mainly invokes `sum`, `param.data.narrow`, `param_data.copy_`, and `loaded_weight.narrow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output_dim`, `shard_offset`, `shard_size`, `param_data`, and `start_idx` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MergedColumnParallelRepeatedLinear.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `sum`、`param.data.narrow`、`param_data.copy_` 以及 `loaded_weight.narrow`，说明该流程会编排底层辅助函数或计算内核。 像 `output_dim`、`shard_offset`、`shard_size`、`param_data` 以及 `start_idx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1645-1655: Class `ColumnParallelBatchedLinear` declaration and shared state
```python
class ColumnParallelBatchedLinear(nn.Module):
    """Column parallel batched linear layer.

    TODO: quantization is not supported yet.
    Args:
        batch: batch dimension of the linear layer.
        input_size: input dimension of the linear layer.
        output_size: output dimension of the linear layer.
        dtype: Data type for the parameters.
    """
```
**EN:** This block introduces class `ColumnParallelBatchedLinear` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Column parallel batched linear layer.
**CN:** 该代码块引入类 `ColumnParallelBatchedLinear`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 1656-1667: `ColumnParallelBatchedLinear` initialization and state setup
```python
    def __init__(
        self, batch: int, input_size: int, output_size: int, dtype: torch.dtype
    ):
        super().__init__()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.weight = nn.Parameter(
            torch.empty(batch, output_size // self.tp_size, input_size, dtype=dtype),
            requires_grad=False,
        )
        setattr(self.weight, "weight_loader", self.weight_loader)
```
**EN:** This block defines `ColumnParallelBatchedLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`, `nn.Parameter`, and `setattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.tp_rank`, `self.tp_size`, and `self.weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ColumnParallelBatchedLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`get_tensor_model_parallel_rank`、`get_tensor_model_parallel_world_size`、`nn.Parameter` 以及 `setattr`，说明该流程会编排底层辅助函数或计算内核。 像 `self.tp_rank`、`self.tp_size` 以及 `self.weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1668-1670: `ColumnParallelBatchedLinear.forward` main forward path
```python
    def forward(self, input: torch.Tensor) -> torch.Tensor:
        return torch.bmm(input, self.weight.transpose(-1, -2))
```
**EN:** This block defines `ColumnParallelBatchedLinear.forward` and contains the main logic for this step. It mainly invokes `torch.bmm` and `self.weight.transpose`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ColumnParallelBatchedLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `torch.bmm` 和 `self.weight.transpose`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 1671-1677: Function `ColumnParallelBatchedLinear.weight_loader` and its core logic
```python
    def weight_loader(
        self, param: Parameter, loaded_weight: torch.Tensor, loaded_shard_id: int
    ) -> torch.Tensor:
        shard_size = self.weight.shape[-2]
        start_idx = self.tp_rank * shard_size
        loaded_weight = loaded_weight.narrow(0, start_idx, shard_size)
        param.data[loaded_shard_id].copy_(loaded_weight)
```
**EN:** This block defines `ColumnParallelBatchedLinear.weight_loader` and contains the main logic for this step. It mainly invokes `loaded_weight.narrow` and `param.data.copy_`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `shard_size`, `start_idx`, and `loaded_weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ColumnParallelBatchedLinear.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `loaded_weight.narrow` 和 `param.data.copy_`，说明该流程会编排底层辅助函数或计算内核。 像 `shard_size`、`start_idx` 以及 `loaded_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `adjust_marlin_shard`, `adjust_bitsandbytes_4bit_shard`, `adjust_scalar_to_fused_array`, `adjust_shard_offsets`, and `LinearBase`. / **主要符号**：核心入口包括 `adjust_marlin_shard`、`adjust_bitsandbytes_4bit_shard`、`adjust_scalar_to_fused_array`、`adjust_shard_offsets` 以及 `LinearBase`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `itertools`, `logging`, `typing.TYPE_CHECKING`, `typing.Dict`, `typing.List`, `typing.Optional`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`itertools`、`logging`、`typing.TYPE_CHECKING`、`typing.Dict`、`typing.List`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch`, `torch.nn`, `torch.nn.parameter.Parameter`, and `torch.nn.parameter.UninitializedParameter` / **第三方依赖**：`torch`、`torch.nn`、`torch.nn.parameter.Parameter` 以及 `torch.nn.parameter.UninitializedParameter`
- **Internal SGLang modules**: `sglang.kernel_api_logging.wrap_method_with_debug_kernel_once`, `sglang.srt.distributed.divide`, `sglang.srt.distributed.get_tensor_model_parallel_rank`, `sglang.srt.distributed.get_tensor_model_parallel_world_size`, `sglang.srt.distributed.get_tp_group`, `sglang.srt.distributed.split_tensor_along_last_dim`, `sglang.srt.distributed.tensor_model_parallel_all_gather`, `sglang.srt.distributed.tensor_model_parallel_all_reduce`, `sglang.srt.distributed.tensor_model_parallel_quant_all_reduce`, `sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`, `sglang.srt.layers.dp_attention.get_attention_tp_group`, and `sglang.srt.layers.dp_attention.is_allocation_symmetric` / **SGLang 内部模块**：`sglang.kernel_api_logging.wrap_method_with_debug_kernel_once`、`sglang.srt.distributed.divide`、`sglang.srt.distributed.get_tensor_model_parallel_rank`、`sglang.srt.distributed.get_tensor_model_parallel_world_size`、`sglang.srt.distributed.get_tp_group`、`sglang.srt.distributed.split_tensor_along_last_dim`、`sglang.srt.distributed.tensor_model_parallel_all_gather`、`sglang.srt.distributed.tensor_model_parallel_all_reduce`、`sglang.srt.distributed.tensor_model_parallel_quant_all_reduce`、`sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`、`sglang.srt.layers.dp_attention.get_attention_tp_group` 以及 `sglang.srt.layers.dp_attention.is_allocation_symmetric`
