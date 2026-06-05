# compressed_tensors_wNa16_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_wNa16_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w Na16 moe quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 w Na16 MoE 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: module imports and setup
```python
from __future__ import annotations

import enum
import logging
from enum import Enum
from typing import TYPE_CHECKING

import torch
from compressed_tensors import CompressionFormat

from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
    NPUW4A16Int4DynamicMoEMethod,
)
from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    WNA16_SUPPORTED_BITS,
    CompressedTensorsMoEScheme,
)
from sglang.srt.layers.quantization.gptq import gptq_marlin_moe_repack
from sglang.srt.layers.quantization.marlin_utils import marlin_moe_permute_scales
from sglang.srt.layers.quantization.utils import replace_parameter
from sglang.srt.utils import get_bool_env_var, is_cuda, is_hip, set_weight_attrs
```
**EN:** This block imports __future__, compressed_tensors, enum, logging, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, compressed_tensors, enum, logging, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 24-31: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
    from sglang.srt.layers.quantization.compressed_tensors.compressed_tensors import (
        CompressedTensorsConfig,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 34-38: initialize __all__
```python
__all__ = [
    "CompressedTensorsWNA16MoE",
    "CompressedTensorsWNA16TritonMoE",
    "NPUCompressedTensorsW4A16Int4DynamicMoE",
]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 40-40: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 41-41: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 43-43: initialize _use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _use_aiter.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _use_aiter。

### Lines 45-46: conditional logic for _use_aiter
```python
if _use_aiter:
    pass
```
**EN:** This block applies conditional logic controlled by `_use_aiter`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_use_aiter` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 49-49: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 52-52: class GPTQMarlinState: definition
```python
class GPTQMarlinState(Enum):
```
**EN:** This block declares `GPTQMarlinState`, a supporting class for the quantization stack. It organizes behaviors such as class state.
**CN:** 该代码块声明 `GPTQMarlinState`，它是量化栈中的支撑类，组织了 class state 等行为。

### Lines 53-53: GPTQMarlinState member: initialize REPACK
```python
    REPACK = enum.auto()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as REPACK.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 REPACK。

### Lines 54-54: GPTQMarlinState member: initialize READY
```python
    READY = enum.auto()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as READY.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 READY。

### Lines 57-58: class CompressedTensorsWNA16MoE: definition
```python
class CompressedTensorsWNA16MoE(CompressedTensorsMoEScheme):
```
**EN:** This block declares `CompressedTensorsWNA16MoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `CompressedTensorsWNA16MoE`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, create_weights, process_weights_after_loading 等行为。

### Lines 59-79: CompressedTensorsWNA16MoE.__init__()
```python
    def __init__(self, quant_config: CompressedTensorsConfig, num_gpu_experts=-1):
        self.quant_config = quant_config
        config = self.quant_config.target_scheme_map["Linear"].get("weights")
        self.num_bits = config.num_bits
        self.packed_factor = 32 // config.num_bits
        self.strategy = config.strategy
        self.group_size = config.group_size
        self.actorder = config.actorder
        assert config.symmetric, "Only symmetric quantization is supported for MoE"

        if not (
            self.quant_config.quant_format == CompressionFormat.pack_quantized.value
            and self.num_bits in WNA16_SUPPORTED_BITS
        ):
            raise ValueError(
                "For Fused MoE layers, only ",
                f"{CompressionFormat.pack_quantized.value} ",
                "is supported for the following bits: ",
                f"{WNA16_SUPPORTED_BITS}",
            )
        self.num_gpu_experts = num_gpu_experts
```
**EN:** This block defines `CompressedTensorsWNA16MoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsWNA16MoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 81-84: CompressedTensorsWNA16MoE.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # ampere and up
        return 80
```
**EN:** This block defines `CompressedTensorsWNA16MoE.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsWNA16MoE.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 86-130: CompressedTensorsWNA16MoE.create_weights() (part 1/4)
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
        # Will transpose the loaded weight along the
        # intermediate and hidden dim sizes. Will
        # shard for TP along the transposed dims
        extra_weight_attrs.update(
            {"is_transposed": True, "quant_method": self.strategy}
        )
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size // self.packed_factor,
                2 * intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_packed", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition // self.packed_factor,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_packed", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # In the case where we have actorder/g_idx,
        # we do not partition the w2 scales
        load_full_w2 = self.actorder and self.group_size != -1

        if load_full_w2:
            w2_scales_size = intermediate_size_per_partition * layer.moe_tp_size
```
**EN:** This segment of `CompressedTensorsWNA16MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsWNA16MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 131-175: CompressedTensorsWNA16MoE.create_weights() (part 2/4)
```python
        else:
            w2_scales_size = intermediate_size_per_partition

        self.is_k_full = (not self.actorder) or layer.moe_tp_size == 1

        if self.strategy == "channel":
            num_groups_w2 = num_groups_w13 = 1
            self.group_size = -1
        else:
            num_groups_w2 = w2_scales_size // self.group_size
            num_groups_w13 = hidden_size // self.group_size

        w13_scale = torch.nn.Parameter(
            torch.ones(
                num_experts,
                num_groups_w13,
                2 * intermediate_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale", w13_scale)
        set_weight_attrs(w13_scale, extra_weight_attrs)

        w2_scale = torch.nn.Parameter(
            torch.ones(num_experts, num_groups_w2, hidden_size, dtype=params_dtype),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_scale", w2_scale)
        set_weight_attrs(w2_scale, extra_weight_attrs)
        set_weight_attrs(w2_scale, {"load_full_w2": load_full_w2})

        w2_weight_shape = torch.nn.Parameter(
            torch.empty(num_experts, 2), requires_grad=False
        )
        layer.register_parameter("w2_weight_shape", w2_weight_shape)
        set_weight_attrs(w2_weight_shape, extra_weight_attrs)
        w13_weight_shape = torch.nn.Parameter(
            torch.empty(num_experts, 2), requires_grad=False
        )

        layer.register_parameter("w13_weight_shape", w13_weight_shape)
        set_weight_attrs(w13_weight_shape, extra_weight_attrs)

        w13_g_idx = torch.nn.Parameter(
```
**EN:** This segment of `CompressedTensorsWNA16MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsWNA16MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 176-220: CompressedTensorsWNA16MoE.create_weights() (part 3/4)
```python
            torch.empty(
                num_experts,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_g_idx", w13_g_idx)
        set_weight_attrs(w13_g_idx, extra_weight_attrs)

        w2_g_idx = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_g_idx", w2_g_idx)
        set_weight_attrs(w2_g_idx, extra_weight_attrs)

        w13_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_g_idx_sort_indices", w13_g_idx_sort_indices)
        set_weight_attrs(w13_g_idx_sort_indices, extra_weight_attrs)

        w2_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_g_idx_sort_indices", w2_g_idx_sort_indices)
        set_weight_attrs(w2_g_idx_sort_indices, extra_weight_attrs)

        layer.a13_scale = None
        layer.a2_scale = None
```
**EN:** This segment of `CompressedTensorsWNA16MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsWNA16MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 221-232: CompressedTensorsWNA16MoE.create_weights() (part 4/4)
```python
        layer.marlin_state = GPTQMarlinState.REPACK

        if not hasattr(layer, "_original_shapes"):
            layer._original_shapes = {}

        # Force record: these are the target GPTQ shapes for rollback.
        layer._original_shapes["w13_weight_packed"] = tuple(w13_weight.shape)
        layer._original_shapes["w2_weight_packed"] = tuple(w2_weight.shape)

        # Also record the shapes of the scales.
        layer._original_shapes["w2_weight_scale"] = tuple(w2_scale.shape)
        layer._original_shapes["w13_weight_scale"] = tuple(w13_scale.shape)
```
**EN:** This segment of `CompressedTensorsWNA16MoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `CompressedTensorsWNA16MoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 234-278: CompressedTensorsWNA16MoE.process_weights_after_loading() (part 1/3)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:

        # Skip if the layer is already converted to Marlin format to prevent double-packing.
        if getattr(layer, "is_marlin_converted", False):
            return

        if not hasattr(layer, "_original_shapes"):
            layer._original_shapes = {}

        def replace_tensor(name, new_t):
            target_attr = getattr(layer, name)

            # Only save if the key doesn't exist to prevent overwriting with Marlin shapes.
            if name not in layer._original_shapes:
                # This is a safety check; `create_weights` usually handles this already.
                layer._original_shapes[name] = tuple(target_attr.shape)

            # It is important to use resize_() here since it ensures
            # the same buffer is reused
            target_attr.resize_(new_t.shape)
            target_attr.copy_(new_t)
            del new_t

        num_experts = layer.w13_weight_g_idx.shape[0]
        device = layer.w13_weight_g_idx.device

        # when running models with grouped act order,
        # resort to g_idx values provided in checkpoint
        if self.actorder == "group":
            w13_g_idx_sort_indices = torch.empty_like(layer.w13_weight_g_idx)
            w2_g_idx_sort_indices = torch.empty_like(layer.w2_weight_g_idx)
            w13_sorted_g_idx = torch.empty_like(layer.w13_weight_g_idx)
            w2_sorted_g_idx = torch.empty_like(layer.w2_weight_g_idx)

            for e in range(num_experts):
                w13_g_idx_sort_indices[e] = torch.argsort(layer.w13_weight_g_idx[e]).to(
                    torch.int32
                )
                w2_g_idx_sort_indices[e] = torch.argsort(layer.w2_weight_g_idx[e]).to(
                    torch.int32
                )
                w13_sorted_g_idx[e] = layer.w13_weight_g_idx[e][
                    w13_g_idx_sort_indices[e]
                ]
                w2_sorted_g_idx[e] = layer.w2_weight_g_idx[e][w2_g_idx_sort_indices[e]]
```
**EN:** This segment of `CompressedTensorsWNA16MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsWNA16MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 279-323: CompressedTensorsWNA16MoE.process_weights_after_loading() (part 2/3)
```python

            replace_parameter(layer, "w13_weight_g_idx", w13_sorted_g_idx)
            replace_parameter(layer, "w2_weight_g_idx", w2_sorted_g_idx)
            replace_parameter(layer, "w13_g_idx_sort_indices", w13_g_idx_sort_indices)
            replace_parameter(layer, "w2_g_idx_sort_indices", w2_g_idx_sort_indices)

        else:
            layer.w13_weight_g_idx = torch.nn.Parameter(
                torch.empty((num_experts, 0), dtype=torch.int32, device=device),
                requires_grad=False,
            )
            layer.w2_weight_g_idx = torch.nn.Parameter(
                torch.empty((num_experts, 0), dtype=torch.int32, device=device),
                requires_grad=False,
            )
            layer.w13_g_idx_sort_indices = torch.nn.Parameter(
                torch.empty((num_experts, 0), dtype=torch.int32, device=device),
                requires_grad=False,
            )
            layer.w2_g_idx_sort_indices = torch.nn.Parameter(
                torch.empty((num_experts, 0), dtype=torch.int32, device=device),
                requires_grad=False,
            )

        marlin_w13_qweight = gptq_marlin_moe_repack(
            layer.w13_weight_packed,
            layer.w13_g_idx_sort_indices,
            layer.w13_weight_packed.shape[1] * self.packed_factor,
            layer.w13_weight_packed.shape[2],
            self.num_bits,
        )
        replace_tensor("w13_weight_packed", marlin_w13_qweight)
        marlin_w2_qweight = gptq_marlin_moe_repack(
            layer.w2_weight_packed,
            layer.w2_g_idx_sort_indices,
            layer.w2_weight_packed.shape[1] * self.packed_factor,
            layer.w2_weight_packed.shape[2],
            self.num_bits,
        )
        replace_tensor("w2_weight_packed", marlin_w2_qweight)
        # Repack scales
        marlin_w13_scales = marlin_moe_permute_scales(
            layer.w13_weight_scale,
            layer.w13_weight_packed.shape[2],
            layer.w13_weight_scale.shape[2],
```
**EN:** This segment of `CompressedTensorsWNA16MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsWNA16MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 324-337: CompressedTensorsWNA16MoE.process_weights_after_loading() (part 3/3)
```python
            self.group_size,
        )
        replace_tensor("w13_weight_scale", marlin_w13_scales)

        marlin_w2_scales = marlin_moe_permute_scales(
            layer.w2_weight_scale,
            layer.w2_weight_scale.shape[1]
            * (self.group_size if self.group_size != -1 else self.packed_factor),
            layer.w2_weight_scale.shape[2],
            self.group_size,
        )
        replace_tensor("w2_weight_scale", marlin_w2_scales)

        layer.is_marlin_converted = True
```
**EN:** This segment of `CompressedTensorsWNA16MoE.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `CompressedTensorsWNA16MoE.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 339-351: CompressedTensorsWNA16MoE.restore_weights_before_loading()
```python
    def restore_weights_before_loading(self, layer: torch.nn.Module):
        """Forcibly resize parameters back to their original shapes (e.g., GPTQ format) before loading weights."""

        if not hasattr(layer, "_original_shapes"):
            return

        for name, orig_shape in layer._original_shapes.items():
            param = getattr(layer, name, None)

            if param is not None and param.shape != orig_shape:
                param.resize_(orig_shape)

        layer.is_marlin_converted = False
```
**EN:** This block defines `CompressedTensorsWNA16MoE.restore_weights_before_loading()`, which handles checkpoint loading or layout conversion.
**CN:** 该代码块定义了 `CompressedTensorsWNA16MoE.restore_weights_before_loading()`，用于处理检查点加载或布局转换。

### Lines 353-357: CompressedTensorsWNA16MoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.MARLIN, moe_runner_config)
```
**EN:** This block defines `CompressedTensorsWNA16MoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsWNA16MoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 359-373: CompressedTensorsWNA16MoE.get_marlin_quant_info()
```python
    def get_marlin_quant_info(self, layer):
        from sglang.srt.layers.moe.moe_runner.marlin import MarlinMoeQuantInfo

        return MarlinMoeQuantInfo(
            w13_qweight=layer.w13_weight_packed,
            w2_qweight=layer.w2_weight_packed,
            w13_scales=layer.w13_weight_scale,
            w2_scales=layer.w2_weight_scale,
            w13_g_idx_sort_indices=getattr(layer, "w13_g_idx_sort_indices", None),
            w2_g_idx_sort_indices=getattr(layer, "w2_g_idx_sort_indices", None),
            weight_bits=self.num_bits,
            w13_g_idx=getattr(layer, "w13_weight_g_idx", None),
            w2_g_idx=getattr(layer, "w2_weight_g_idx", None),
            is_k_full=self.is_k_full,
        )
```
**EN:** This block defines `CompressedTensorsWNA16MoE.get_marlin_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsWNA16MoE.get_marlin_quant_info()`，用于为调用方获取或计算派生值。

### Lines 375-423: CompressedTensorsWNA16MoE.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        from sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe import (
            fused_marlin_moe,
        )
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        assert (
            self.moe_runner_config.activation == "silu"
        ), "Only SiLU activation is supported."

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        topk_weights, topk_ids, router_logits = topk_output

        # Get expert_map for EP support
        expert_map = None
        global_num_experts = -1
        if hasattr(layer, "dispatcher") and hasattr(
            layer.dispatcher, "local_expert_mapping"
        ):
            expert_map = layer.dispatcher.local_expert_mapping
            if expert_map is not None:
                global_num_experts = self.moe_runner_config.num_experts

        output = fused_marlin_moe(
            x,
            layer.w13_weight_packed,
            layer.w2_weight_packed,
            layer.w13_weight_scale,
            layer.w2_weight_scale,
            router_logits,
            topk_weights,
            topk_ids,
            global_num_experts=global_num_experts,
            expert_map=expert_map,
            g_idx1=layer.w13_weight_g_idx,
            g_idx2=layer.w2_weight_g_idx,
            sort_indices1=layer.w13_g_idx_sort_indices,
            sort_indices2=layer.w2_g_idx_sort_indices,
            num_bits=self.num_bits,
            is_k_full=self.is_k_full,
            routed_scaling_factor=self.moe_runner_config.routed_scaling_factor,
        )
        return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `CompressedTensorsWNA16MoE.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsWNA16MoE.apply_weights()`，用于将量化计算应用到运行时输入上。

### Lines 426-433: class CompressedTensorsWNA16TritonMoE: definition
```python
class CompressedTensorsWNA16TritonMoE(CompressedTensorsWNA16MoE):
    """ROCm/HIP-compatible W4A16 MoE method using Triton kernels instead of Marlin.

    Inherits weight creation from CompressedTensorsWNA16MoE but converts
    weights to the uint8-packed format expected by the Triton fused MoE kernel
    instead of the Marlin-specific format.
    """
```
**EN:** This block declares `CompressedTensorsWNA16TritonMoE`, a supporting class for the quantization stack. It organizes behaviors such as process_weights_after_loading, create_moe_runner, get_triton_quant_info, apply_weights.
**CN:** 该代码块声明 `CompressedTensorsWNA16TritonMoE`，它是量化栈中的支撑类，组织了 process_weights_after_loading, create_moe_runner, get_triton_quant_info, apply_weights 等行为。

### Lines 434-460: CompressedTensorsWNA16TritonMoE.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if getattr(layer, "is_triton_converted", False):
            return

        num_experts = layer.w13_weight_packed.shape[0]

        # Convert w13 weights: [E, K//8, N] int32 -> [E, N, K//2] uint8
        w13 = layer.w13_weight_packed.data
        w13 = w13.transpose(1, 2).contiguous().view(torch.uint8)
        layer.w13_weight_packed = torch.nn.Parameter(w13, requires_grad=False)

        # Convert w2 weights: [E, K//8, N] int32 -> [E, N, K//2] uint8
        w2 = layer.w2_weight_packed.data
        w2 = w2.transpose(1, 2).contiguous().view(torch.uint8)
        layer.w2_weight_packed = torch.nn.Parameter(w2, requires_grad=False)

        # Convert w13 scales: [E, K//group_size, N] -> [E, N, K//group_size]
        w13_scale = layer.w13_weight_scale.data
        w13_scale = w13_scale.transpose(1, 2).contiguous()
        layer.w13_weight_scale = torch.nn.Parameter(w13_scale, requires_grad=False)

        # Convert w2 scales: [E, K//group_size, N] -> [E, N, K//group_size]
        w2_scale = layer.w2_weight_scale.data
        w2_scale = w2_scale.transpose(1, 2).contiguous()
        layer.w2_weight_scale = torch.nn.Parameter(w2_scale, requires_grad=False)

        layer.is_triton_converted = True
```
**EN:** This block defines `CompressedTensorsWNA16TritonMoE.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `CompressedTensorsWNA16TritonMoE.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 462-466: CompressedTensorsWNA16TritonMoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.TRITON, moe_runner_config)
```
**EN:** This block defines `CompressedTensorsWNA16TritonMoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsWNA16TritonMoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 468-478: CompressedTensorsWNA16TritonMoE.get_triton_quant_info()
```python
    def get_triton_quant_info(self, layer):
        from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo

        return TritonMoeQuantInfo(
            w13_weight=layer.w13_weight_packed,
            w2_weight=layer.w2_weight_packed,
            use_int4_w4a16=True,
            w13_scale=layer.w13_weight_scale,
            w2_scale=layer.w2_weight_scale,
            block_shape=[0, self.group_size],
        )
```
**EN:** This block defines `CompressedTensorsWNA16TritonMoE.get_triton_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsWNA16TritonMoE.get_triton_quant_info()`，用于为调用方获取或计算派生值。

### Lines 480-490: CompressedTensorsWNA16TritonMoE.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: "StandardDispatchOutput",
    ) -> "CombineInput":
        assert (
            self.moe_runner_config.activation == "silu"
        ), "Only SiLU activation is supported."

        quant_info = self.get_triton_quant_info(layer)
        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `CompressedTensorsWNA16TritonMoE.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsWNA16TritonMoE.apply_weights()`，用于将量化计算应用到运行时输入上。

### Lines 493-494: class NPUCompressedTensorsW4A16Int4DynamicMoE: definition
```python
class NPUCompressedTensorsW4A16Int4DynamicMoE(CompressedTensorsMoEScheme):
```
**EN:** This block declares `NPUCompressedTensorsW4A16Int4DynamicMoE`, a scheme class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `NPUCompressedTensorsW4A16Int4DynamicMoE`，它是量化栈中的执行方案类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 495-507: NPUCompressedTensorsW4A16Int4DynamicMoE.__init__()
```python
    def __init__(self, quantization_config) -> None:
        self.pack_factor = 8  # weight dtype is int4,  but use int32 to create
        target = (
            "MoEGMM" if "MoEGMM" in quantization_config.target_scheme_map else "Linear"
        )
        if target in quantization_config.target_scheme_map:
            self.group_size = quantization_config.target_scheme_map[target][
                "weights"
            ].group_size
        else:
            self.group_size = 128

        self.kernel = NPUW4A16Int4DynamicMoEMethod()
```
**EN:** This block defines `NPUCompressedTensorsW4A16Int4DynamicMoE.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A16Int4DynamicMoE.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 512-556: NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights() (part 1/3)
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ) -> None:
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        self.num_experts = num_experts
        if (
            extra_weight_attrs.get(
                "moe_intermediate_size", intermediate_size_per_partition
            )
            // intermediate_size_per_partition
            > 1
        ):
            quant_method = FusedMoeWeightScaleSupported.GROUP.value
        else:
            quant_method = FusedMoeWeightScaleSupported.CHANNEL.value
        extra_weight_attrs.update({"quant_method": quant_method})
        # weight
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // self.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)
        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // self.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
```
**EN:** This segment of `NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 557-601: NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights() (part 2/3)
```python
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # scale
        weight_scale_dtype = torch.bfloat16
        w13_weight_scale = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // self.group_size,
                dtype=weight_scale_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        set_weight_attrs(w13_weight_scale, extra_weight_attrs)
        w2_weight_scale = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // self.group_size,
                dtype=weight_scale_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_scale", w2_weight_scale)
        set_weight_attrs(w2_weight_scale, extra_weight_attrs)

        # offset
        w13_weight_offset = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // self.group_size,
                dtype=weight_scale_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_offset", w13_weight_offset)
        set_weight_attrs(w13_weight_offset, extra_weight_attrs)

        w2_weight_offset = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // self.group_size,
```
**EN:** This segment of `NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 602-619: NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights() (part 3/3)
```python
                dtype=weight_scale_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight_offset", w2_weight_offset)
        set_weight_attrs(w2_weight_offset, extra_weight_attrs)

        w13_weight_shape = torch.nn.Parameter(
            torch.empty(num_experts, 2), requires_grad=False
        )
        layer.register_parameter("w13_weight_shape", w13_weight_shape)
        set_weight_attrs(w13_weight_shape, extra_weight_attrs)

        w2_weight_shape = torch.nn.Parameter(
            torch.empty(num_experts, 2), requires_grad=False
        )
        layer.register_parameter("w2_weight_shape", w2_weight_shape)
        set_weight_attrs(w2_weight_shape, extra_weight_attrs)
```
**EN:** This segment of `NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `NPUCompressedTensorsW4A16Int4DynamicMoE.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 621-622: NPUCompressedTensorsW4A16Int4DynamicMoE.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        self.kernel.process_weights_after_loading(layer)
```
**EN:** This block defines `NPUCompressedTensorsW4A16Int4DynamicMoE.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A16Int4DynamicMoE.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 624-627: NPUCompressedTensorsW4A16Int4DynamicMoE.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `NPUCompressedTensorsW4A16Int4DynamicMoE.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A16Int4DynamicMoE.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 629-635: NPUCompressedTensorsW4A16Int4DynamicMoE.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        return self.kernel.apply(layer, dispatch_output)
```
**EN:** This block defines `NPUCompressedTensorsW4A16Int4DynamicMoE.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A16Int4DynamicMoE.apply_weights()`，用于将量化计算应用到运行时输入上。

### Lines 637-653: NPUCompressedTensorsW4A16Int4DynamicMoE.apply_without_routing_weights()
```python
    def apply_without_routing_weights(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        return self.kernel.apply_without_routing_weights(
            layer,
            hidden_states,
            hidden_states_scale,
            group_list_type,
            group_list,
            output_dtype,
        )
```
**EN:** This block defines `NPUCompressedTensorsW4A16Int4DynamicMoE.apply_without_routing_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `NPUCompressedTensorsW4A16Int4DynamicMoE.apply_without_routing_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `GPTQMarlinState`: A supporting class that structures file-level quantization behavior. / `GPTQMarlinState` 是一个支撑类，用于组织该文件中的量化行为。
- `CompressedTensorsWNA16MoE`: A scheme class that structures file-level quantization behavior. / `CompressedTensorsWNA16MoE` 是一个执行方案类，用于组织该文件中的量化行为。
- `CompressedTensorsWNA16TritonMoE`: A supporting class that structures file-level quantization behavior. / `CompressedTensorsWNA16TritonMoE` 是一个支撑类，用于组织该文件中的量化行为。
- `NPUCompressedTensorsW4A16Int4DynamicMoE`: A scheme class that structures file-level quantization behavior. / `NPUCompressedTensorsW4A16Int4DynamicMoE` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `compressed_tensors`, `enum`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe`, `sglang.srt.layers.moe.moe_runner.marlin`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.compressed_tensors.compressed_tensors`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.layers.quantization.gptq`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
