# compressed_tensors_w8a8_fp8_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_w8a8_fp8_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w8a8 fp8 moe quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 W8A8 FP8 MoE 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

import torch
from compressed_tensors.quantization import QuantizationStrategy

from sglang.srt.distributed import get_tensor_model_parallel_world_size
from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.flashinfer_trtllm import (
    FlashInferTrtllmFp8MoeQuantInfo,
)
from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
from sglang.srt.layers.moe.utils import (
    get_moe_a2a_backend,
    get_moe_runner_backend,
    get_moe_weight_sizes,
)
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    CompressedTensorsMoEScheme,
)
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz, scaled_fp8_quant
from sglang.srt.layers.quantization.fp8_utils import normalize_e4m3fn_to_e4m3fnuz
from sglang.srt.layers.quantization.utils import (
    all_close_1d,
    per_tensor_dequantize,
    swap_w13_to_w31,
)
from sglang.srt.utils import get_bool_env_var, is_hip, set_weight_attrs
```
**EN:** This block imports __future__, aiter.ops.shuffle, compressed_tensors.quantization, logging, sglang.srt.distributed, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner.aiter and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, aiter.ops.shuffle, compressed_tensors.quantization, logging, sglang.srt.distributed, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.moe_runner.aiter 等依赖，并为当前量化实现准备模块命名空间。

### Lines 32-37: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.fused_moe_triton import FusedMoE
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 39-39: initialize __all__
```python
__all__ = ["CompressedTensorsW8A8Fp8MoE"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 41-41: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 42-42: initialize _use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _use_aiter.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _use_aiter。

### Lines 44-45: conditional logic for _use_aiter
```python
if _use_aiter:
    from aiter.ops.shuffle import shuffle_weight
```
**EN:** This block applies conditional logic controlled by `_use_aiter`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_use_aiter` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 48-48: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 51-52: class CompressedTensorsW8A8Fp8MoE: definition
```python
class CompressedTensorsW8A8Fp8MoE(CompressedTensorsMoEScheme):
```
**EN:** This block declares `CompressedTensorsW8A8Fp8MoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `CompressedTensorsW8A8Fp8MoE`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, create_weights, process_weights_after_loading 等行为。

### Lines 53-79: CompressedTensorsW8A8Fp8MoE.__init__()
```python
    def __init__(self, weight_quant, input_quant):
        self.weight_quant = weight_quant
        self.input_quant = input_quant
        self.use_flashinfer_trtllm = get_moe_runner_backend().is_flashinfer_trtllm()

        per_tensor = (
            self.weight_quant.strategy == QuantizationStrategy.TENSOR
            and self.input_quant.strategy == QuantizationStrategy.TENSOR
        )
        per_channel = (
            self.weight_quant.strategy == QuantizationStrategy.CHANNEL
            and self.input_quant.strategy == QuantizationStrategy.TOKEN
        )
        if not (per_tensor or per_channel):
            assert self.weight_quant.strategy == QuantizationStrategy.BLOCK
            self.weight_block_size = self.weight_quant.block_structure
            assert self.weight_quant.dynamic is not None
        else:
            self.weight_block_size = None
        self.block_quant = self.weight_block_size is not None

        self.static_input_scales = not self.input_quant.dynamic
        if self.static_input_scales and per_channel:
            raise ValueError(
                "For FP8 Fused MoE layer, we require either per tensor or "
                "channelwise, dynamic per token quantization."
            )
```
**EN:** This block defines `CompressedTensorsW8A8Fp8MoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Fp8MoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 81-84: CompressedTensorsW8A8Fp8MoE.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # ampere and up
        return 80
```
**EN:** This block defines `CompressedTensorsW8A8Fp8MoE.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Fp8MoE.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 86-130: CompressedTensorsW8A8Fp8MoE.create_weights() (part 1/4)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        params_dtype = torch.float8_e4m3fn

        if self.block_quant:
            assert self.weight_block_size is not None
            layer.weight_block_size = self.weight_block_size
            tp_size = get_tensor_model_parallel_world_size()
            block_n, block_k = (
                self.weight_block_size[0],
                self.weight_block_size[1],
            )
            # NOTE: To ensure proper alignment of the block-wise quantization
            # scales, the output_size of the weights for both the gate and up
            # layers must be divisible by block_n.
            # Required by column parallel or enabling merged weights
            if intermediate_size_per_partition % block_n != 0:
                raise ValueError(
                    f"The output_size of gate's and up's weight = "
                    f"{intermediate_size_per_partition} is not divisible by "
                    f"weight quantization block_n = {block_n}."
                )
            if tp_size > 1 and intermediate_size_per_partition % block_k != 0:
                # Required by row parallel
                raise ValueError(
                    f"The input_size of down's weight = "
                    f"{intermediate_size_per_partition} is not divisible by "
                    f"weight quantization block_k = {block_k}."
                )

        w13_up_dim, w2_down_dim, weight_padded = get_moe_weight_sizes(
            intermediate_size_per_partition,
            is_aiter_moe=_use_aiter,
            is_concat=True,
            is_packed=False,
        )
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 131-175: CompressedTensorsW8A8Fp8MoE.create_weights() (part 2/4)
```python

        extra_weight_attrs.update(
            {"weight_padded": weight_padded},
        )

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                w13_up_dim,
                hidden_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                w2_down_dim,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # WEIGHT_SCALES
        # per-tensor quantization
        if self.weight_quant.strategy == QuantizationStrategy.TENSOR:
            # Allocate 2 scales for w1 and w3 respectively.
            # They will be combined to a single scale after weight loading.
            w13_weight_scale = torch.nn.Parameter(
                torch.ones(num_experts, 2, dtype=torch.float32), requires_grad=False
            )
            w2_weight_scale = torch.nn.Parameter(
                torch.ones(num_experts, dtype=torch.float32), requires_grad=False
            )
            weight_quant_method = FusedMoeWeightScaleSupported.TENSOR.value
        elif self.weight_quant.strategy == QuantizationStrategy.CHANNEL:
            w13_weight_scale = torch.nn.Parameter(
                torch.ones(
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 176-220: CompressedTensorsW8A8Fp8MoE.create_weights() (part 3/4)
```python
                    num_experts,
                    w13_up_dim,
                    1,
                    dtype=torch.float32,
                ),
                requires_grad=False,
            )
            w2_weight_scale = torch.nn.Parameter(
                torch.ones(num_experts, hidden_size, 1, dtype=torch.float32),
                requires_grad=False,
            )
            weight_quant_method = FusedMoeWeightScaleSupported.CHANNEL.value
        elif self.weight_quant.strategy == QuantizationStrategy.BLOCK:
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
            weight_quant_method = FusedMoeWeightScaleSupported.BLOCK.value
        else:
            raise ValueError(
                f"Unsupported weight quantization strategy: {self.weight_quant.strategy}"
            )

        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
        # Add the quantization method used (per tensor/grouped/channel)
        # to ensure the weight scales are loaded in properly
        extra_weight_attrs.update({"quant_method": weight_quant_method})
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 221-239: CompressedTensorsW8A8Fp8MoE.create_weights() (part 4/4)
```python
        # INPUT_SCALES
        if self.static_input_scales:
            assert (
                self.input_quant.strategy == QuantizationStrategy.TENSOR
            ), "Only per-tensor quantization is supported for static input scales"
            w13_input_scale = torch.nn.Parameter(
                torch.ones(num_experts, dtype=torch.float32), requires_grad=False
            )
            layer.register_parameter("w13_input_scale", w13_input_scale)
            set_weight_attrs(w13_input_scale, extra_weight_attrs)

            w2_input_scale = torch.nn.Parameter(
                torch.ones(num_experts, dtype=torch.float32), requires_grad=False
            )
            layer.register_parameter("w2_input_scale", w2_input_scale)
            set_weight_attrs(w2_input_scale, extra_weight_attrs)
        else:
            layer.w13_input_scale = None
            layer.w2_input_scale = None
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 241-285: CompressedTensorsW8A8Fp8MoE.process_weights_after_loading() (part 1/3)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module | FusedMoE) -> None:
        # Fp8 moe kernels require a single activation scale.
        # We take the max of all the scales in case they differ.
        if self.static_input_scales:
            if layer.w13_input_scale is None or layer.w2_input_scale is None:
                raise ValueError(
                    "QuantConfig has static quantization, but found "
                    "activation scales are None."
                )
            if not all_close_1d(layer.w13_input_scale) or not all_close_1d(
                layer.w2_input_scale
            ):
                logger.warning(
                    "Found input_scales that are not equal for "
                    "fp8 MoE layer. Using the maximum across experts "
                    "for each layer."
                )
            layer.w13_input_scale = torch.nn.Parameter(
                layer.w13_input_scale.max(), requires_grad=False
            )
            layer.w2_input_scale = torch.nn.Parameter(
                layer.w2_input_scale.max(), requires_grad=False
            )

        if is_fp8_fnuz():
            # Normalize the weights and scales
            w13_weight, w13_weight_scale, w13_input_scale = (
                normalize_e4m3fn_to_e4m3fnuz(
                    layer.w13_weight, layer.w13_weight_scale, layer.w13_input_scale
                )
            )
            w2_weight, w2_weight_scale, w2_input_scale = normalize_e4m3fn_to_e4m3fnuz(
                layer.w2_weight, layer.w2_weight_scale, layer.w2_input_scale
            )
            # Reset the parameter
            layer.w13_weight = torch.nn.Parameter(w13_weight, requires_grad=False)
            layer.w13_weight_scale = torch.nn.Parameter(
                w13_weight_scale, requires_grad=False
            )
            if w13_input_scale is not None:
                layer.w13_input_scale = torch.nn.Parameter(
                    w13_input_scale, requires_grad=False
                )
            layer.w2_weight = torch.nn.Parameter(w2_weight, requires_grad=False)
            layer.w2_weight_scale = torch.nn.Parameter(
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 286-330: CompressedTensorsW8A8Fp8MoE.process_weights_after_loading() (part 2/3)
```python
                w2_weight_scale, requires_grad=False
            )
            if w2_input_scale is not None:
                layer.w2_input_scale = torch.nn.Parameter(
                    w2_input_scale, requires_grad=False
                )
        if self.weight_quant.strategy == QuantizationStrategy.TENSOR:
            # Fp8 moe kernel needs single weight scale for w13 per expert.
            # We take the max then dequant and requant each expert.
            assert layer.w13_weight_scale is not None
            shard_size = layer.intermediate_size_per_partition
            max_w13_scales = layer.w13_weight_scale.max(dim=1).values
            for expert_id in range(layer.num_local_experts):
                start = 0
                for shard_id in range(2):
                    dq_weight = per_tensor_dequantize(
                        layer.w13_weight[expert_id][start : start + shard_size, :],
                        layer.w13_weight_scale[expert_id][shard_id],
                    )
                    (
                        layer.w13_weight[expert_id][start : start + shard_size, :],
                        _,
                    ) = scaled_fp8_quant(dq_weight, max_w13_scales[expert_id])

                    start += shard_size

            layer.w13_weight_scale = torch.nn.Parameter(
                max_w13_scales, requires_grad=False
            )

        if self.weight_quant.strategy == QuantizationStrategy.CHANNEL and _use_aiter:
            with torch.no_grad():
                # Pre-shuffle weights
                layer.w13_weight = torch.nn.Parameter(
                    shuffle_weight(layer.w13_weight.data, (16, 16)),
                    requires_grad=False,
                )
                torch.cuda.empty_cache()
                layer.w2_weight = torch.nn.Parameter(
                    shuffle_weight(layer.w2_weight.data, (16, 16)),
                    requires_grad=False,
                )
                torch.cuda.empty_cache()

        if (
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 331-341: CompressedTensorsW8A8Fp8MoE.process_weights_after_loading() (part 3/3)
```python
            self.weight_quant.strategy == QuantizationStrategy.BLOCK
            and self.use_flashinfer_trtllm
        ):
            layer.w13_weight = torch.nn.Parameter(
                swap_w13_to_w31(layer.w13_weight.data),
                requires_grad=False,
            )
            layer.w13_weight_scale = torch.nn.Parameter(
                swap_w13_to_w31(layer.w13_weight_scale.data),
                requires_grad=False,
            )
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 343-367: CompressedTensorsW8A8Fp8MoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        moe_runner_backend = get_moe_runner_backend()
        if moe_runner_backend.is_auto():
            if (
                _use_aiter
                and self.weight_quant.strategy == QuantizationStrategy.CHANNEL
                and get_moe_a2a_backend().supports_aiter()
            ):
                moe_runner_backend = MoeRunnerBackend.AITER
            else:
                moe_runner_backend = MoeRunnerBackend.TRITON

        if (
            moe_runner_backend.is_aiter()
            or moe_runner_backend.is_triton()
            or moe_runner_backend.is_flashinfer_trtllm()
            or moe_runner_backend.is_flashinfer_trtllm_routed()
        ):
            self.runner = MoeRunner(moe_runner_backend, moe_runner_config)
        else:
            # TODO(cwan): refactor other backends
            pass
```
**EN:** This block defines `CompressedTensorsW8A8Fp8MoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsW8A8Fp8MoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 369-413: CompressedTensorsW8A8Fp8MoE.apply_weights() (part 1/2)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        moe_runner_config = self.moe_runner_config

        if self.runner.runner_backend.is_aiter():
            from sglang.srt.layers.moe.moe_runner.aiter import (
                AiterMoeQuantInfo,
                AiterQuantType,
            )

            assert not moe_runner_config.no_combine, "unsupported"
            quant_info = AiterMoeQuantInfo(
                w13_weight=layer.w13_weight,
                w2_weight=layer.w2_weight,
                quant_type=AiterQuantType.PER_TOKEN,
                w13_scale=layer.w13_weight_scale,
                w2_scale=layer.w2_weight_scale,
                a13_scale=layer.w13_input_scale,
                a2_scale=layer.w2_input_scale,
            )
            return self.runner.run(dispatch_output, quant_info)
        elif self.weight_quant.strategy == QuantizationStrategy.BLOCK:
            if self.use_flashinfer_trtllm:
                from sglang.srt.layers.moe.moe_runner.flashinfer_trtllm import (
                    get_activation_type,
                )

                activation_type = get_activation_type(moe_runner_config.activation)
                quant_info = FlashInferTrtllmFp8MoeQuantInfo(
                    w13_weight=layer.w13_weight,
                    w2_weight=layer.w2_weight,
                    global_num_experts=layer.num_experts,
                    local_expert_offset=layer.moe_ep_rank * layer.num_local_experts,
                    local_num_experts=layer.num_local_experts,
                    intermediate_size=layer.w2_weight.shape[2],
                    routing_method_type=layer.routing_method_type,
                    block_quant=self.block_quant,
                    weight_block_k=self.weight_block_size[1],
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.apply_weights()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.apply_weights()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 414-442: CompressedTensorsW8A8Fp8MoE.apply_weights() (part 2/2)
```python
                    w13_weight_scale_inv=layer.w13_weight_scale,
                    w2_weight_scale_inv=layer.w2_weight_scale,
                    activation_type=activation_type,
                )
            else:
                quant_info = TritonMoeQuantInfo(
                    w13_weight=layer.w13_weight,
                    w2_weight=layer.w2_weight,
                    use_fp8_w8a8=True,
                    w13_scale=layer.w13_weight_scale,
                    w2_scale=layer.w2_weight_scale,
                    a13_scale=layer.w13_input_scale,
                    a2_scale=layer.w2_input_scale,
                    block_shape=self.weight_block_size,
                )
            return self.runner.run(dispatch_output, quant_info)
        else:
            quant_info = TritonMoeQuantInfo(
                w13_weight=layer.w13_weight,
                w2_weight=layer.w2_weight,
                use_fp8_w8a8=True,
                per_channel_quant=self.weight_quant.strategy
                == QuantizationStrategy.CHANNEL,
                w13_scale=layer.w13_weight_scale,
                w2_scale=layer.w2_weight_scale,
                a13_scale=layer.w13_input_scale,
                a2_scale=layer.w2_input_scale,
            )
            return self.runner.run(dispatch_output, quant_info)
```
**EN:** This segment of `CompressedTensorsW8A8Fp8MoE.apply_weights()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `CompressedTensorsW8A8Fp8MoE.apply_weights()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

## Key Concepts / 关键概念
- `CompressedTensorsW8A8Fp8MoE`: A scheme class that structures file-level quantization behavior. / `CompressedTensorsW8A8Fp8MoE` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `aiter.ops.shuffle`, `compressed_tensors.quantization`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.aiter`, `sglang.srt.layers.moe.moe_runner.flashinfer_trtllm`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.utils`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
