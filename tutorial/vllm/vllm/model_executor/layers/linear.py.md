# linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/linear.py`
- **Repository**: vllm-project/vllm
- **Purpose**: linear projection, quantization, and tensor-parallel utilities / 线性投影、量化与张量并行工具

## Line-by-Line Analysis / 逐行分析
### Lines 4-64 — imports and setup
```python
import itertools
from abc import abstractmethod

import torch
from torch.nn.parameter import Parameter, UninitializedParameter

import vllm.envs as envs
from vllm.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    split_tensor_along_last_dim,
    tensor_model_parallel_all_gather,
    tensor_model_parallel_all_reduce,
)
from vllm.logger import init_logger
from vllm.model_executor.custom_op import PluggableLayer
from vllm.model_executor.layers.batch_invariant import (
    linear_batch_invariant,
)
from vllm.model_executor.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from vllm.model_executor.layers.utils import (
    dispatch_unquantized_gemm,
)
from vllm.model_executor.parameter import (
    BasevLLMParameter,
    BlockQuantScaleParameter,
    ModelWeightParameter,
    PackedColumnParameter,
    PackedvLLMParameter,
    PerTensorScaleParameter,
    RowvLLMParameter,
)
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform

logger = init_logger(__name__)

WEIGHT_LOADER_V2_SUPPORTED = [
    "UnquantizedLinearMethod",
    "CompressedTensorsLinearMethod",
    "CompressedTensorsLinearTransformMethod",
    "AWQMarlinLinearMethod",
    "AWQLinearMethod",
    "AutoGPTQLinearMethod",
    "Fp8LinearMethod",
    "MarlinLinearMethod",
    "GPTQMarlin24LinearMethod",
    "TPUInt8LinearMethod",
    "FBGEMMFp8LinearMethod",
    "ModelOptFp8LinearMethod",
    "ModelOptFp8PcPtLinearMethod",
    "ModelOptFp8PbWoLinearMethod",
# ... omitted for brevity ...
    "HummingLinearMethod",
]
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`, `WEIGHT_LOADER_V2_SUPPORTED`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`, `WEIGHT_LOADER_V2_SUPPORTED`。

### Lines 67-70 — function `register_weight_loader_v2_supported_method`
```python
def register_weight_loader_v2_supported_method(cls):
    """Decorator to register a LinearMethod as supporting weight_loader_v2."""
    WEIGHT_LOADER_V2_SUPPORTED.append(cls.__name__)
    return cls
```
**EN:** This function defines `register_weight_loader_v2_supported_method`. Decorator to register a LinearMethod as supporting weight_loader_v2. Key calls include `WEIGHT_LOADER_V2_SUPPORTED.append`.
**CN:** 该函数定义 `register_weight_loader_v2_supported_method`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `WEIGHT_LOADER_V2_SUPPORTED.append`。

### Lines 73-82 — function `adjust_marlin_shard`
```python
def adjust_marlin_shard(
    param: Parameter,
    shard_size: int,
    shard_offset: int,
) -> tuple[int, int]:
    marlin_tile_size: int | None = getattr(param, "marlin_tile_size", None)
    if marlin_tile_size is None:
        return shard_size, shard_offset

    return shard_size * marlin_tile_size, shard_offset * marlin_tile_size
```
**EN:** This function defines `adjust_marlin_shard`. It provides one of the file's main runtime building blocks. The main inputs are `param`, `shard_size`, `shard_offset`. Key calls include `getattr`. It writes or updates `marlin_tile_size`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `adjust_marlin_shard`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `param`, `shard_size`, `shard_offset`。 关键调用包括 `getattr`。 它会写入或更新 `marlin_tile_size`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 85-94 — function `adjust_block_scale_shard`
```python
def adjust_block_scale_shard(
    weight_block_size: tuple[int, ...] | None,
    shard_size: int,
    shard_offset: int,
) -> tuple[int, int]:
    assert weight_block_size is not None
    block_n = weight_block_size[0]
    shard_offset = (shard_offset + block_n - 1) // block_n
    shard_size = (shard_size + block_n - 1) // block_n
    return shard_size, shard_offset
```
**EN:** This function defines `adjust_block_scale_shard`. It provides one of the file's main runtime building blocks. The main inputs are `weight_block_size`, `shard_size`, `shard_offset`. It writes or updates `block_n`, `shard_offset`, `shard_size`.
**CN:** 该函数定义 `adjust_block_scale_shard`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `weight_block_size`, `shard_size`, `shard_offset`。 它会写入或更新 `block_n`, `shard_offset`, `shard_size`。

### Lines 97-111 — function `adjust_bitsandbytes_4bit_shard`
```python
def adjust_bitsandbytes_4bit_shard(
    param: Parameter,
    shard_offsets: dict[str, tuple[int, int]],
    loaded_shard_id: str,
) -> tuple[int, int]:
    """Adjust the quantization offsets and sizes for BitsAndBytes sharding."""

    total, _ = shard_offsets["total"]
    orig_offset, orig_size = shard_offsets[loaded_shard_id]

    quantized_total = param.data.shape[0]
    quantized_offset = orig_offset * quantized_total // total
    quantized_size = orig_size * quantized_total // total

    return quantized_size, quantized_offset
```
**EN:** This function defines `adjust_bitsandbytes_4bit_shard`. Adjust the quantization offsets and sizes for BitsAndBytes sharding. The main inputs are `param`, `shard_offsets`, `loaded_shard_id`. It writes or updates `total`, `_`, `orig_offset`, `orig_size`, `quantized_total`, `quantized_offset`.
**CN:** 该函数定义 `adjust_bitsandbytes_4bit_shard`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `param`, `shard_offsets`, `loaded_shard_id`。 它会写入或更新 `total`, `_`, `orig_offset`, `orig_size`, `quantized_total`, `quantized_offset`。

### Lines 114-138 — function `adjust_scalar_to_fused_array`
```python
def adjust_scalar_to_fused_array(
    param_data: torch.Tensor,
    loaded_weight: torch.Tensor,
    shard_id: int | str,
) -> tuple[torch.Tensor, torch.Tensor]:
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

    return param_data[shard_id], loaded_weight
```
**EN:** This function defines `adjust_scalar_to_fused_array`. For fused modules (QKV and MLP) we have an array of length N that holds 1 scale for each "logical" matrix. The main inputs are `param_data`, `loaded_weight`, `shard_id`. Key calls include `isinstance`, `len`, `ValueError`. It writes or updates `qkv_idxs`, `shard_id`, `loaded_weight`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `adjust_scalar_to_fused_array`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `param_data`, `loaded_weight`, `shard_id`。 关键调用包括 `isinstance`, `len`, `ValueError`。 它会写入或更新 `qkv_idxs`, `shard_id`, `loaded_weight`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 141-179 — class `LinearMethodBase`
```python
class LinearMethodBase(QuantizeMethodBase):
    """Base class for different (maybe quantized) linear methods."""

    @abstractmethod
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
# ... omitted for brevity ...
        Expects create_weights to have been called before on the layer."""
        raise NotImplementedError
```
**EN:** This class defines `LinearMethodBase`. It inherits from `QuantizeMethodBase`. Base class for different (maybe quantized) linear methods. Important methods include `apply`, `create_weights`.
**CN:** 该类定义了 `LinearMethodBase`。 它继承自 `QuantizeMethodBase`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `apply`, `create_weights`。

### Lines 170-179 — method `LinearMethodBase.apply`
```python
    @abstractmethod
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Apply the weights in layer to the input tensor.
        Expects create_weights to have been called before on the layer."""
        raise NotImplementedError
```
**EN:** This method defines `apply`. Apply the weights in layer to the input tensor. The main inputs are `layer`, `x`, `bias`.
**CN:** 该方法定义 `apply`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `layer`, `x`, `bias`。

### Lines 144-168 — method `LinearMethodBase.create_weights`
```python
    @abstractmethod
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        """Create weights for a linear layer.
           The weights will be set as attributes of the layer.

        Args:
            layer: The layer that is using the LinearMethodBase factory.
            input_size_per_partition: Size of the weight input dim on rank X.
            output_partition_sizes: Sizes of the output dim of each logical
                weight on rank X. E.g., output_partition_sizes for QKVLinear
                is a list contains the width of Wq, Wk, Wv on rank X.
            input_size: Size of the input dim of the weight across all ranks.
            output_size: Size of the output dim of the weight across all ranks.
            params_dtype: Datatype of the parameters.
        """
        raise NotImplementedError
```
**EN:** This method defines `create_weights`. Create weights for a linear layer. The main inputs are `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`.
**CN:** 该方法定义 `create_weights`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`。

### Lines 182-228 — class `UnquantizedLinearMethod`
```python
class UnquantizedLinearMethod(LinearMethodBase):
    """Linear method without quantization."""

    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        # This method creates unquantized linear weights.
# ... omitted for brevity ...
            return linear_batch_invariant(x, layer.weight, bias)
        return dispatch_unquantized_gemm()(layer, x, layer.weight, bias)
```
**EN:** This class defines `UnquantizedLinearMethod`. It inherits from `LinearMethodBase`. Linear method without quantization. Important methods include `apply`, `create_weights`, `process_weights_after_loading`. Key calls include `extra_weight_attrs.pop`, `ModelWeightParameter`, `layer.register_parameter`, `set_weight_attrs`, `current_platform.is_cpu`, `dispatch_unquantized_gemm`. It writes or updates `weight_loader`, `weight`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `UnquantizedLinearMethod`。 它继承自 `LinearMethodBase`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `apply`, `create_weights`, `process_weights_after_loading`。 关键调用包括 `extra_weight_attrs.pop`, `ModelWeightParameter`, `layer.register_parameter`, `set_weight_attrs`, `current_platform.is_cpu`, `dispatch_unquantized_gemm`。 它会写入或更新 `weight_loader`, `weight`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 220-228 — method `UnquantizedLinearMethod.apply`
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if envs.VLLM_BATCH_INVARIANT and current_platform.is_cuda_alike():
            return linear_batch_invariant(x, layer.weight, bias)
        return dispatch_unquantized_gemm()(layer, x, layer.weight, bias)
```
**EN:** This method defines `apply`. It provides one of the file's main runtime building blocks. The main inputs are `layer`, `x`, `bias`. Key calls include `dispatch_unquantized_gemm`, `current_platform.is_cuda_alike`, `linear_batch_invariant`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `layer`, `x`, `bias`。 关键调用包括 `dispatch_unquantized_gemm`, `current_platform.is_cuda_alike`, `linear_batch_invariant`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 185-212 — method `UnquantizedLinearMethod.create_weights`
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        # This method creates unquantized linear weights.
        # The weights are not quantized, and they are not sharded.
        # The amount of memory allocated for the weights is
        # sum(output_partition_sizes) * input_size_per_partition.
        weight_loader = extra_weight_attrs.pop("weight_loader")
        weight = ModelWeightParameter(
            data=torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition,
                dtype=params_dtype,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )

        layer.register_parameter("weight", weight)
        set_weight_attrs(weight, extra_weight_attrs)
```
**EN:** This method defines `create_weights`. It provides one of the file's main runtime building blocks. The main inputs are `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`. Key calls include `extra_weight_attrs.pop`, `ModelWeightParameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.empty`, `sum`. It writes or updates `weight_loader`, `weight`.
**CN:** 该方法定义 `create_weights`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`。 关键调用包括 `extra_weight_attrs.pop`, `ModelWeightParameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.empty`, `sum`。 它会写入或更新 `weight_loader`, `weight`。

### Lines 214-218 — method `UnquantizedLinearMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if current_platform.is_cpu():
            from vllm.model_executor.layers.utils import dispatch_cpu_unquantized_gemm

            dispatch_cpu_unquantized_gemm(layer, remove_weight=True)
```
**EN:** This method defines `process_weights_after_loading`. It provides one of the file's main runtime building blocks. The main inputs are `layer`. Key calls include `current_platform.is_cpu`, `dispatch_cpu_unquantized_gemm`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `process_weights_after_loading`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `layer`。 关键调用包括 `current_platform.is_cpu`, `dispatch_cpu_unquantized_gemm`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 231-287 — class `LinearBase`
```python
class LinearBase(PluggableLayer):
    """Base linear layer.

    Args:
        input_size: input dimension of the linear layer.
        output_size: output dimension of the linear layer.
        skip_bias_add: If true, skip adding bias but instead return it.
        params_dtype: Data type for the parameters.
        quant_config: Quantization configure.
        prefix: Prefix for parameter names.
        return_bias: If true, return bias together with outputs in forward pass.
        disable_tp: If true, tensor parallelism will be disabled for this layer.
    """

# ... omitted for brevity ...
                param.tp_rank = self.tp_rank
                param.tp_size = self.tp_size
```
**EN:** This class defines `LinearBase`. It inherits from `PluggableLayer`. Base linear layer. Important methods include `__init__`, `update_param_tp_status`. Key calls include `super.__init__`, `self.parameters`, `torch.get_default_dtype`, `UnquantizedLinearMethod`, `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`. It writes or updates `input_size`, `output_size`, `has_bias`, `skip_bias_add`, `params_dtype`, `quant_config`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `LinearBase`。 它继承自 `PluggableLayer`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `update_param_tp_status`。 关键调用包括 `super.__init__`, `self.parameters`, `torch.get_default_dtype`, `UnquantizedLinearMethod`, `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`。 它会写入或更新 `input_size`, `output_size`, `has_bias`, `skip_bias_add`, `params_dtype`, `quant_config`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Linear projection, quantization, and tensor-parallel utilities / [CN] 线性投影、量化与张量并行工具
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `register_weight_loader_v2_supported_method`, `adjust_marlin_shard`, `adjust_block_scale_shard`, `adjust_bitsandbytes_4bit_shard` / [CN] 核心符号：`register_weight_loader_v2_supported_method`, `adjust_marlin_shard`, `adjust_block_scale_shard`, `adjust_bitsandbytes_4bit_shard`

## Dependencies / 依赖关系
- **External**: `itertools`, `abc`, `torch`, `torch.nn.parameter` / **外部依赖**: `itertools`, `abc`, `torch`, `torch.nn.parameter`
- **Internal**: `vllm.envs`, `vllm.distributed`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.batch_invariant`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.utils`, `vllm.model_executor.parameter`, `vllm.model_executor.utils`, `vllm.platforms` / **内部依赖**: `vllm.envs`, `vllm.distributed`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.batch_invariant`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.utils`, `vllm.model_executor.parameter`, `vllm.model_executor.utils`, `vllm.platforms`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
