# blockwise_int8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/blockwise_int8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for blockwise int8 quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 分块 INT8 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/model_executor/layers/quantization/fp8.py

from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any, Dict, List, Optional

import torch
from torch.nn import Module

from sglang.srt.distributed import get_tensor_model_parallel_world_size
from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
from sglang.srt.layers.parameter import BlockQuantScaleParameter, ModelWeightParameter
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.int8_utils import apply_w8a8_block_int8_linear
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.layers.quantization.utils import is_layer_skipped
from sglang.srt.utils import set_weight_attrs
```
**EN:** This block imports __future__, logging, torch, torch.nn, sglang.srt.distributed, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, torch, torch.nn, sglang.srt.distributed, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton 等依赖，并为当前量化实现准备模块命名空间。

### Lines 28-32: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 34-34: initialize ACTIVATION_SCHEMES
```python
ACTIVATION_SCHEMES = ["static", "dynamic"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as ACTIVATION_SCHEMES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 ACTIVATION_SCHEMES。

### Lines 36-36: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 39-41: class BlockInt8Config: definition
```python
class BlockInt8Config(QuantizationConfig):
    """Config class for INT8."""
```
**EN:** This block declares `BlockInt8Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_name, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `BlockInt8Config`，它是量化栈中的配置类，组织了 __init__, get_name, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 42-72: BlockInt8Config.__init__()
```python
    def __init__(
        self,
        is_checkpoint_int8_serialized: bool = False,
        activation_scheme: str = "dynamic",
        ignored_layers: Optional[List[str]] = None,
        weight_block_size: List[int] = None,
    ) -> None:
        self.is_checkpoint_int8_serialized = is_checkpoint_int8_serialized
        if is_checkpoint_int8_serialized:
            logger.warning(
                "Detected int8 checkpoint. Please note that the "
                "format is experimental and subject to change."
            )
        if activation_scheme not in ACTIVATION_SCHEMES:
            raise ValueError(f"Unsupported activation scheme {activation_scheme}")
        self.activation_scheme = activation_scheme
        self.ignored_layers = ignored_layers or []
        if weight_block_size is not None:
            if not is_checkpoint_int8_serialized:
                raise ValueError(
                    f"The block-wise quantization only supports int8-serialized checkpoint for now."
                )
            if len(weight_block_size) != 2:
                raise ValueError(
                    f"The quantization block size of weight must have 2 dimensions, but got {len(weight_block_size)} dimensions."
                )
            if activation_scheme != "dynamic":
                raise ValueError(
                    f"The block-wise quantization only supports dynamic activation scheme for now, but got {activation_scheme} activation scheme."
                )
        self.weight_block_size = weight_block_size
```
**EN:** This block defines `BlockInt8Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `BlockInt8Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 74-76: BlockInt8Config.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "blockwise_int8"
```
**EN:** This block defines `BlockInt8Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BlockInt8Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 78-80: BlockInt8Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.bfloat16, torch.half]
```
**EN:** This block defines `BlockInt8Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BlockInt8Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 82-84: BlockInt8Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This block defines `BlockInt8Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BlockInt8Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 86-88: BlockInt8Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return []
```
**EN:** This block defines `BlockInt8Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BlockInt8Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 90-102: BlockInt8Config.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> BlockInt8Config:
        quant_method = cls.get_from_keys(config, ["quant_method"])
        is_checkpoint_int8_serialized = "int8" in quant_method
        activation_scheme = cls.get_from_keys(config, ["activation_scheme"])
        ignored_layers = cls.get_from_keys_or(config, ["ignored_layers"], None)
        weight_block_size = cls.get_from_keys_or(config, ["weight_block_size"], None)
        return cls(
            is_checkpoint_int8_serialized=is_checkpoint_int8_serialized,
            activation_scheme=activation_scheme,
            ignored_layers=ignored_layers,
            weight_block_size=weight_block_size,
        )
```
**EN:** This block defines `BlockInt8Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `BlockInt8Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 104-116: BlockInt8Config.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if isinstance(layer, LinearBase):
            if is_layer_skipped(prefix, self.ignored_layers):
                return UnquantizedLinearMethod()
            return BlockInt8LinearMethod(self)
        elif isinstance(layer, FusedMoE):
            return BlockInt8MoEMethod(self)
        return None
```
**EN:** This block defines `BlockInt8Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BlockInt8Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 118-119: BlockInt8Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `BlockInt8Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BlockInt8Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 122-133: class BlockInt8LinearMethod: definition
```python
class BlockInt8LinearMethod(LinearMethodBase):
    """Linear method for INT8.
    Supports loading INT8 checkpoints with static weight scale and
    dynamic activation scale.

    Limitations:
    Only support block-wise int8 quantization and int8 checkpoint

    Args:
        quant_config: The quantization config.
    """
```
**EN:** This block declares `BlockInt8LinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `BlockInt8LinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 134-137: BlockInt8LinearMethod.__init__()
```python
    def __init__(self, quant_config: BlockInt8Config):
        self.quant_config = quant_config
        assert self.quant_config.weight_block_size is not None
        assert self.quant_config.is_checkpoint_int8_serialized
```
**EN:** This block defines `BlockInt8LinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `BlockInt8LinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 139-183: BlockInt8LinearMethod.create_weights() (part 1/2)
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
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        tp_size = get_tensor_model_parallel_world_size()

        block_n, block_k = (
            self.quant_config.weight_block_size[0],
            self.quant_config.weight_block_size[1],
        )
        # Required by row parallel
        if tp_size > 1 and input_size // input_size_per_partition == tp_size:
            if input_size_per_partition % block_k != 0:
                raise ValueError(
                    f"Weight input_size_per_partition = "
                    f"{input_size_per_partition} is not divisible by "
                    f"weight quantization block_k = {block_k}."
                )
        # Required by column parallel or enabling merged weights
        if (tp_size > 1 and output_size // output_size_per_partition == tp_size) or len(
            output_partition_sizes
        ) > 1:
            for output_partition_size in output_partition_sizes:
                if output_partition_size % block_n != 0:
                    raise ValueError(
                        f"Weight output_partition_size = "
                        f"{output_partition_size} is not divisible by "
                        f"weight quantization block_n = {block_n}."
                    )

        layer.logical_widths = output_partition_sizes

        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype
```
**EN:** This segment of `BlockInt8LinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `BlockInt8LinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 184-218: BlockInt8LinearMethod.create_weights() (part 2/2)
```python
        # WEIGHT
        weight_dtype = (
            torch.int8
            if self.quant_config.is_checkpoint_int8_serialized
            else params_dtype
        )

        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition, input_size_per_partition, dtype=weight_dtype
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        # WEIGHT SCALE

        scale = BlockQuantScaleParameter(
            data=torch.empty(
                (output_size_per_partition + block_n - 1) // block_n,
                (input_size_per_partition + block_k - 1) // block_k,
                dtype=torch.float32,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        scale[:] = torch.finfo(torch.float32).min
        layer.register_parameter("weight_scale_inv", scale)

        # INPUT ACTIVATION SCALE
        assert self.quant_config.activation_scheme == "dynamic"
        layer.register_parameter("input_scale", None)
```
**EN:** This segment of `BlockInt8LinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `BlockInt8LinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 220-226: BlockInt8LinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        # Block quant doesn't need to process weights after loading
        # Use torch Parameter to avoid cuda graph capturing issue
        layer.weight = torch.nn.Parameter(layer.weight.data, requires_grad=False)
        layer.weight_scale_inv = torch.nn.Parameter(
            layer.weight_scale_inv.data, requires_grad=False
        )
```
**EN:** This block defines `BlockInt8LinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `BlockInt8LinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 228-241: BlockInt8LinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return apply_w8a8_block_int8_linear(
            input=x,
            weight=layer.weight,
            block_size=self.quant_config.weight_block_size,
            weight_scale=layer.weight_scale_inv,
            input_scale=None,
            bias=bias,
        )
```
**EN:** This block defines `BlockInt8LinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `BlockInt8LinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 244-255: class BlockInt8MoEMethod: definition
```python
class BlockInt8MoEMethod(FusedMoEMethodBase):
    """MoE method for INT8.
    Supports loading INT8 checkpoints with static weight scale and
    dynamic activation scale.

    Limitations:
    Only support block-wise int8 quantization and int8 checkpoint

    Args:
        quant_config: The quantization config.
    """
```
**EN:** This block declares `BlockInt8MoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `BlockInt8MoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 256-259: BlockInt8MoEMethod.__init__()
```python
    def __init__(self, quant_config: BlockInt8Config):
        self.quant_config = quant_config
        assert self.quant_config.weight_block_size is not None
        assert self.quant_config.is_checkpoint_int8_serialized
```
**EN:** This block defines `BlockInt8MoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `BlockInt8MoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 261-305: BlockInt8MoEMethod.create_weights() (part 1/3)
```python
    def create_weights(
        self,
        layer: Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        if self.quant_config.is_checkpoint_int8_serialized:
            params_dtype = torch.int8
        tp_size = get_tensor_model_parallel_world_size()

        block_n, block_k = (
            self.quant_config.weight_block_size[0],
            self.quant_config.weight_block_size[1],
        )
        # NOTE(HandH1998): To ensure proper alignment of the block-wise quantization scales, the output_size of the weights for both the gate and up layers must be divisible by block_n.
        # Required by column parallel or enabling merged weights
        if intermediate_size_per_partition % block_n != 0:
            raise ValueError(
                f"The output_size of gate's and up's weight = "
                f"{intermediate_size_per_partition} is not divisible by "
                f"weight quantization block_n = {block_n}."
            )
        if tp_size > 1:
            # Required by row parallel
            if intermediate_size_per_partition % block_k != 0:
                raise ValueError(
                    f"The input_size of down's weight = "
                    f"{intermediate_size_per_partition} is not divisible by "
                    f"weight quantization block_k = {block_k}."
                )

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
```
**EN:** This segment of `BlockInt8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `BlockInt8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 306-350: BlockInt8MoEMethod.create_weights() (part 2/3)
```python
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # WEIGHT_SCALES
        w13_weight_scale = torch.nn.Parameter(
            torch.ones(
                num_experts,
                2 * ((intermediate_size_per_partition + block_n - 1) // block_n),
                (hidden_size + block_k - 1) // block_k,
                dtype=torch.float32,
            ),
            requires_grad=False,
        )
        w2_weight_scale = torch.nn.Parameter(
            torch.ones(
                num_experts,
                (hidden_size + block_n - 1) // block_n,
                (intermediate_size_per_partition + block_k - 1) // block_k,
                dtype=torch.float32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale_inv", w13_weight_scale)
        layer.register_parameter("w2_weight_scale_inv", w2_weight_scale)

        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.BLOCK.value}
        )
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        # INPUT_SCALES
```
**EN:** This segment of `BlockInt8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `BlockInt8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 351-353: BlockInt8MoEMethod.create_weights() (part 3/3)
```python
        assert self.quant_config.activation_scheme == "dynamic"
        layer.w13_input_scale = None
        layer.w2_input_scale = None
```
**EN:** This segment of `BlockInt8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `BlockInt8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 355-357: BlockInt8MoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        # Block quant doesn't need to process weights after loading
        return
```
**EN:** This block defines `BlockInt8MoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `BlockInt8MoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 359-363: BlockInt8MoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.TRITON, moe_runner_config)
```
**EN:** This block defines `BlockInt8MoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `BlockInt8MoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 365-375: BlockInt8MoEMethod.get_triton_quant_info()
```python
    def get_triton_quant_info(self, layer: torch.nn.Module) -> TritonMoeQuantInfo:
        return TritonMoeQuantInfo(
            w13_weight=layer.w13_weight,
            w2_weight=layer.w2_weight,
            use_int8_w8a8=True,
            w13_scale=layer.w13_weight_scale_inv,
            w2_scale=layer.w2_weight_scale_inv,
            a13_scale=layer.w13_input_scale,
            a2_scale=layer.w2_input_scale,
            block_shape=self.quant_config.weight_block_size,
        )
```
**EN:** This block defines `BlockInt8MoEMethod.get_triton_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `BlockInt8MoEMethod.get_triton_quant_info()`，用于为调用方获取或计算派生值。

### Lines 377-385: BlockInt8MoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        quant_info = self.get_triton_quant_info(layer)

        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `BlockInt8MoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `BlockInt8MoEMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `ACTIVATION_SCHEMES`: A module-level constant or registry. / `ACTIVATION_SCHEMES`：模块级常量或注册表。
- `BlockInt8Config`: A configuration class that structures file-level quantization behavior. / `BlockInt8Config` 是一个配置类，用于组织该文件中的量化行为。
- `BlockInt8LinearMethod`: A runtime method class that structures file-level quantization behavior. / `BlockInt8LinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `BlockInt8MoEMethod`: A runtime method class that structures file-level quantization behavior. / `BlockInt8MoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `torch`, `torch.nn`, `typing`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.int8_utils`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
