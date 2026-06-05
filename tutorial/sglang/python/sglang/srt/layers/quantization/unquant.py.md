# unquant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/unquant.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime method classes that apply unquant quantized weights during inference. / 该模块实现了推理期运行方法类，用于应用 非量化 量化权重。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: module imports and setup
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, List, Optional
```
**EN:** This block imports __future__, aiter.ops.shuffle, aiter.tuned_gemm, flashinfer.fused_moe, sglang.srt.environ, sglang.srt.hardware_backend.npu.utils, sglang.srt.layers.activation, sglang.srt.layers.amx_utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, aiter.ops.shuffle, aiter.tuned_gemm, flashinfer.fused_moe, sglang.srt.environ, sglang.srt.hardware_backend.npu.utils, sglang.srt.layers.activation, sglang.srt.layers.amx_utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 6-6: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 8-42: module imports and setup
```python
import torch
import torch.nn.functional as F
from torch.nn.parameter import Parameter

from sglang.srt.environ import envs
from sglang.srt.layers.amx_utils import (
    CPUQuantMethod,
    _amx_process_weight_after_loading,
)
from sglang.srt.layers.moe import (
    MoeRunner,
    MoeRunnerBackend,
    MoeRunnerConfig,
    get_deepep_mode,
    get_moe_a2a_backend,
    get_moe_runner_backend,
)
from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizeMethodBase,
)
from sglang.srt.layers.utils import MultiPlatformOp, copy_or_rebind_param
from sglang.srt.utils import (
    cpu_has_amx_support,
    get_bool_env_var,
    is_cpu,
    is_hip,
    is_npu,
    next_power_of_2,
    set_weight_attrs,
    use_intel_amx_backend,
    use_intel_xpu_backend,
)
```
**EN:** This block imports __future__, aiter.ops.shuffle, aiter.tuned_gemm, flashinfer.fused_moe, sglang.srt.environ, sglang.srt.hardware_backend.npu.utils, sglang.srt.layers.activation, sglang.srt.layers.amx_utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, aiter.ops.shuffle, aiter.tuned_gemm, flashinfer.fused_moe, sglang.srt.environ, sglang.srt.hardware_backend.npu.utils, sglang.srt.layers.activation, sglang.srt.layers.amx_utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 44-48: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 51-51: initialize _is_cpu_amx_available
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cpu_amx_available.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cpu_amx_available。

### Lines 52-52: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 53-53: initialize _is_cpu
```python
_is_cpu = is_cpu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cpu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cpu。

### Lines 54-54: initialize _is_npu
```python
_is_npu = is_npu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_npu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_npu。

### Lines 55-55: initialize _use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _use_aiter.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _use_aiter。

### Lines 57-59: conditional logic for _use_aiter
```python
if _use_aiter:
    from aiter.ops.shuffle import shuffle_weight
    from aiter.tuned_gemm import tgemm
```
**EN:** This block applies conditional logic controlled by `_use_aiter`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_use_aiter` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 61-62: conditional logic for _is_npu
```python
if _is_npu:
    from sglang.srt.hardware_backend.npu.utils import npu_format_cast
```
**EN:** This block applies conditional logic controlled by `_is_npu`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_npu` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 64-68: guarded import or fallback path
```python
try:
    from flashinfer.fused_moe import cutlass_fused_moe as flashinfer_cutlass_fused_moe
    from flashinfer.fused_moe.core import ActivationType
except ImportError:
    flashinfer_cutlass_fused_moe = None
```
**EN:** This block uses exception handling to provide a fallback implementation when optional functionality is unavailable.
**CN:** 该代码块通过异常处理在可选功能不可用时提供后备实现。

### Lines 71-73: class UnquantizedEmbeddingMethod: definition
```python
class UnquantizedEmbeddingMethod(QuantizeMethodBase):
    """Unquantized method for embeddings."""
```
**EN:** This block declares `UnquantizedEmbeddingMethod`, a runtime method class for the quantization stack. It organizes behaviors such as create_weights, apply, embedding.
**CN:** 该代码块声明 `UnquantizedEmbeddingMethod`，它是量化栈中的运行方法类，组织了 create_weights, apply, embedding 等行为。

### Lines 74-95: UnquantizedEmbeddingMethod.create_weights()
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
        """Create weights for embedding layer."""
        weight = Parameter(
            torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        set_weight_attrs(weight, {"input_dim": 1, "output_dim": 0})
        layer.register_parameter("weight", weight)
        set_weight_attrs(weight, extra_weight_attrs)
```
**EN:** This block defines `UnquantizedEmbeddingMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `UnquantizedEmbeddingMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 97-103: UnquantizedEmbeddingMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return F.linear(x, layer.weight, bias)
```
**EN:** This block defines `UnquantizedEmbeddingMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `UnquantizedEmbeddingMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 105-106: UnquantizedEmbeddingMethod.embedding()
```python
    def embedding(self, layer: torch.nn.Module, input_: torch.Tensor) -> torch.Tensor:
        return F.embedding(input_, layer.weight)
```
**EN:** This block defines `UnquantizedEmbeddingMethod.embedding()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `UnquantizedEmbeddingMethod.embedding()`，用于实现量化栈中的可复用模块逻辑。

### Lines 109-111: class UnquantizedLinearMethod: definition
```python
class UnquantizedLinearMethod(LinearMethodBase):
    """Linear method without quantization."""
```
**EN:** This block declares `UnquantizedLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `UnquantizedLinearMethod`，它是量化栈中的运行方法类，组织了 create_weights, process_weights_after_loading, apply 等行为。

### Lines 112-132: UnquantizedLinearMethod.create_weights()
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
        weight = Parameter(
            torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        set_weight_attrs(weight, {"input_dim": 1, "output_dim": 0})
        layer.register_parameter("weight", weight)
        set_weight_attrs(weight, extra_weight_attrs)
```
**EN:** This block defines `UnquantizedLinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `UnquantizedLinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 134-136: UnquantizedLinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if _is_cpu and _is_cpu_amx_available:
            _amx_process_weight_after_loading(layer, ["weight"])
```
**EN:** This block defines `UnquantizedLinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `UnquantizedLinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 138-161: UnquantizedLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if use_intel_amx_backend(layer):
            x_shapes = x.shape
            if len(x_shapes) == 3:
                x = x.view(-1, x.shape[-1])
            output = torch.ops.sgl_kernel.weight_packed_linear(
                x,
                layer.weight,
                bias,
                True,  # is_vnni
            )
            if len(x_shapes) == 3:
                output = output.view(x_shapes[0], x_shapes[1], -1)
            return output

        elif _use_aiter and type(layer.weight.data) is torch.Tensor:
            return tgemm.mm(x, layer.weight, bias, otype=x.dtype)

        return F.linear(x, layer.weight, bias)
```
**EN:** This block defines `UnquantizedLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `UnquantizedLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 164-166: class UnquantizedFusedMoEMethod: definition
```python
class UnquantizedFusedMoEMethod(FusedMoEMethodBase, MultiPlatformOp):
    """MoE method without quantization."""
```
**EN:** This block declares `UnquantizedFusedMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, maybe_restore_flashinfer_trtllm_bf16_weight_shape_for_load.
**CN:** 该代码块声明 `UnquantizedFusedMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, maybe_restore_flashinfer_trtllm_bf16_weight_shape_for_load 等行为。

### Lines 167-179: UnquantizedFusedMoEMethod.__init__()
```python
    def __init__(
        self,
        use_triton_kernels: bool = False,
        use_flashinfer_trtllm_moe: bool = False,
        use_deep_gemm: bool = False,
    ):
        super().__init__()
        self.use_flashinfer_cutlass = get_moe_runner_backend().is_flashinfer_cutlass()
        self.use_triton_kernels = use_triton_kernels
        self.with_bias = False
        self.use_flashinfer_trtllm_moe = use_flashinfer_trtllm_moe
        self.use_deep_gemm = use_deep_gemm
        self._cache_permute_indices = dict({})
```
**EN:** This block defines `UnquantizedFusedMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 181-237: UnquantizedFusedMoEMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        with_bias: bool = False,
        **extra_weight_attrs,
    ):
        self.with_bias = with_bias

        # Fused gate_up_proj (column parallel)
        w13_up_dim = (
            2 * intermediate_size_per_partition
            if layer.moe_runner_config.is_gated
            else intermediate_size_per_partition
        )
        w13_weight_n, w13_weight_k = (w13_up_dim, hidden_size)
        if self.use_triton_kernels:
            w13_weight_n, w13_weight_k = w13_weight_k, w13_weight_n
        w13_weight = torch.nn.Parameter(
            torch.empty(num_experts, w13_weight_n, w13_weight_k, dtype=params_dtype),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        if self.with_bias:
            w13_weight_bias = torch.nn.Parameter(
                torch.empty(num_experts, w13_up_dim, dtype=torch.float32),
                requires_grad=False,
            )
            layer.register_parameter("w13_weight_bias", w13_weight_bias)
            set_weight_attrs(w13_weight_bias, extra_weight_attrs)

        # down_proj (row parallel)
        w2_weight_n, w2_weight_k = (
            hidden_size,
            intermediate_size_per_partition,
        )
        if self.use_triton_kernels:
            w2_weight_n, w2_weight_k = w2_weight_k, w2_weight_n
        w2_weight = torch.nn.Parameter(
            torch.empty(num_experts, w2_weight_n, w2_weight_k, dtype=params_dtype),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        if self.with_bias:
            w2_weight_bias = torch.nn.Parameter(
                torch.empty(num_experts, hidden_size, dtype=torch.float32),
                requires_grad=False,
            )
            layer.register_parameter("w2_weight_bias", w2_weight_bias)
            set_weight_attrs(w2_weight_bias, extra_weight_attrs)
```
**EN:** This block defines `UnquantizedFusedMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 239-283: UnquantizedFusedMoEMethod.process_weights_after_loading() (part 1/3)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        _should_use_aiter_moe = _use_aiter and (
            get_moe_runner_backend().is_auto() or get_moe_runner_backend().is_aiter()
        )
        if _should_use_aiter_moe:
            copy_or_rebind_param(
                layer, "w13_weight", shuffle_weight(layer.w13_weight.data, (16, 16))
            )
            torch.cuda.empty_cache()
            copy_or_rebind_param(
                layer, "w2_weight", shuffle_weight(layer.w2_weight.data, (16, 16))
            )
            torch.cuda.empty_cache()

        # Pack weight for get better performance on CPU
        if _is_cpu and _is_cpu_amx_available:
            _amx_process_weight_after_loading(layer, ["w13_weight", "w2_weight"])

        if (
            self.use_deep_gemm
            and layer.w13_weight.dtype == torch.bfloat16
            and get_moe_a2a_backend().is_deepep()
            and get_deepep_mode().enable_low_latency()
            and not _is_npu
            and not _is_hip
            and hasattr(layer, "dispatcher")
        ):
            layer.dispatcher.set_quant_config({"dispatcher_output_dtype": "bf16"})

        # Reorder rows of W1 for fused gated activation
        if self.use_flashinfer_trtllm_moe:
            from flashinfer.fused_moe.core import (
                _maybe_get_cached_w3_w1_permute_indices,
                convert_to_block_layout,
                get_w2_permute_indices_with_cache,
            )

            # w1 and w3 have been swapped, so we don't need do that here
            epilogue_tile_m = 128
            block_k = 128
            old_shape_w13 = layer.w13_weight.data[0].shape
            old_shape_w2 = layer.w2_weight.data[0].shape
            new_shape_w13 = None
            new_shape_w2 = None
            for i in range(layer.num_local_experts):
```
**EN:** This segment of `UnquantizedFusedMoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `UnquantizedFusedMoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 284-328: UnquantizedFusedMoEMethod.process_weights_after_loading() (part 2/3)
```python
                permute_indices = _maybe_get_cached_w3_w1_permute_indices(
                    self._cache_permute_indices,
                    layer.w13_weight.data[i].view(torch.uint8),
                    epilogue_tile_m,
                )
                tmp_weights1 = (
                    layer.w13_weight.data[i]
                    .clone()
                    .view(torch.uint8)[permute_indices.to(layer.w13_weight.data.device)]
                    .contiguous()
                )

                permute_indices = get_w2_permute_indices_with_cache(
                    self._cache_permute_indices,
                    layer.w2_weight.data[i].view(torch.uint8),
                    epilogue_tile_m,
                )
                tmp_weights2 = (
                    layer.w2_weight.data[i]
                    .clone()
                    .view(torch.uint8)[permute_indices.to(layer.w2_weight.data.device)]
                    .contiguous()
                )

                tmp_weights1 = convert_to_block_layout(
                    tmp_weights1.view(torch.uint8), block_k
                )
                tmp_weights2 = convert_to_block_layout(
                    tmp_weights2.view(torch.uint8), block_k
                )

                new_shape_w13 = tmp_weights1.view(torch.bfloat16).shape
                new_shape_w2 = tmp_weights2.view(torch.bfloat16).shape
                layer.w13_weight.data[i] = (
                    tmp_weights1.view(torch.bfloat16)
                    .contiguous()
                    .reshape(old_shape_w13)
                )
                layer.w2_weight.data[i] = (
                    tmp_weights2.view(torch.bfloat16).contiguous().reshape(old_shape_w2)
                )

            layer.w13_weight.data = layer.w13_weight.data.reshape(
                layer.num_local_experts, *new_shape_w13
            )
```
**EN:** This segment of `UnquantizedFusedMoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `UnquantizedFusedMoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 329-341: UnquantizedFusedMoEMethod.process_weights_after_loading() (part 3/3)
```python
            layer.w2_weight.data = layer.w2_weight.data.reshape(
                layer.num_local_experts, *new_shape_w2
            )

        if _is_npu:
            for weight_name in ["w13_weight", "w2_weight"]:
                weight = getattr(layer, weight_name)
                origin_weight = weight.data.transpose(1, 2)
                new_weight = origin_weight.contiguous()
                origin_weight.untyped_storage().resize_(0)
                weight.data = npu_format_cast(new_weight)

        return
```
**EN:** This segment of `UnquantizedFusedMoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `UnquantizedFusedMoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 343-383: UnquantizedFusedMoEMethod.maybe_restore_flashinfer_trtllm_bf16_weight_shape_for_load()
```python
    def maybe_restore_flashinfer_trtllm_bf16_weight_shape_for_load(
        self,
        layer: torch.nn.Module,
        param: torch.nn.Parameter,
        weight_name: str,
    ) -> None:
        """Restore canonical BF16 MoE load shapes before hot weight copy.

        The flashinfer TRT-LLM BF16 postprocess reshapes expert weights into
        block layout. During weight update, checkpoint tensors are in
        canonical layout and need a temporary shape restore for copy.
        """
        if not get_moe_runner_backend().is_flashinfer_trtllm_routed():
            return

        expected_shape = None
        if weight_name.endswith(".experts.w13_weight"):
            w13_rows = (
                2 * layer.intermediate_size_per_partition
                if layer.moe_runner_config.is_gated
                else layer.intermediate_size_per_partition
            )
            expected_shape = (layer.num_local_experts, w13_rows, layer.hidden_size)
        elif weight_name.endswith(".experts.w2_weight"):
            expected_shape = (
                layer.num_local_experts,
                layer.hidden_size,
                layer.intermediate_size_per_partition,
            )

        if expected_shape is None or tuple(param.data.shape) == expected_shape:
            return

        expected_numel = expected_shape[0] * expected_shape[1] * expected_shape[2]
        if param.data.numel() != expected_numel:
            raise RuntimeError(
                f"Cannot restore flashinfer TRT-LLM BF16 MoE weight shape for {weight_name}: "
                f"current shape={tuple(param.data.shape)}, expected shape={expected_shape}."
            )

        param.data = param.data.reshape(expected_shape)
```
**EN:** This block defines `UnquantizedFusedMoEMethod.maybe_restore_flashinfer_trtllm_bf16_weight_shape_for_load()`, which handles checkpoint loading or layout conversion.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.maybe_restore_flashinfer_trtllm_bf16_weight_shape_for_load()`，用于处理检查点加载或布局转换。

### Lines 385-413: UnquantizedFusedMoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        if self.use_flashinfer_trtllm_moe:
            backend = (
                MoeRunnerBackend.FLASHINFER_TRTLLM_ROUTED
                if get_moe_runner_backend().is_flashinfer_trtllm_routed()
                else MoeRunnerBackend.FLASHINFER_TRTLLM
            )
        elif self.use_deep_gemm:
            backend = MoeRunnerBackend.DEEP_GEMM
        elif self.use_triton_kernels:
            backend = MoeRunnerBackend.TRITON_KERNELS
        else:
            backend = MoeRunnerBackend.TRITON
        self.runner = MoeRunner(backend, moe_runner_config)

        # Separate runner so CK-shape errors fall back to self.runner on every call.
        self._aiter_runner: Optional[MoeRunner] = None
        if (
            _use_aiter
            and (
                get_moe_runner_backend().is_auto()
                or get_moe_runner_backend().is_aiter()
            )
            and get_moe_a2a_backend().supports_aiter()
        ):
            self._aiter_runner = MoeRunner(MoeRunnerBackend.AITER, moe_runner_config)
```
**EN:** This block defines `UnquantizedFusedMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 415-418: UnquantizedFusedMoEMethod.load_up_proj_weight_first()
```python
    @property
    def load_up_proj_weight_first(self) -> bool:
        # FlashInfer CUTLASS kernel assumes [Up, Gate] Proj as W13
        return self.use_flashinfer_cutlass
```
**EN:** This block defines `UnquantizedFusedMoEMethod.load_up_proj_weight_first()`, which handles checkpoint loading or layout conversion.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.load_up_proj_weight_first()`，用于处理检查点加载或布局转换。

### Lines 420-428: UnquantizedFusedMoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        return self.forward(
            layer=layer,
            dispatch_output=dispatch_output,
        )
```
**EN:** This block defines `UnquantizedFusedMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 430-474: UnquantizedFusedMoEMethod.forward_cuda() (part 1/3)
```python
    def forward_cuda(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states

        moe_runner_config = self.moe_runner_config

        backend = self.runner.runner_backend
        if backend.is_triton_kernels():
            from sglang.srt.layers.moe.moe_runner.triton_kernels import (
                TritonKernelsQuantInfo,
            )

            quant_info = TritonKernelsQuantInfo(
                w13_weight=layer.w13_weight,
                w2_weight=layer.w2_weight,
                w13_bias=getattr(layer, "w13_weight_bias", None),
                w2_bias=getattr(layer, "w2_weight_bias", None),
            )
            return self.runner.run(dispatch_output, quant_info)
        elif self.runner.runner_backend.is_deep_gemm():
            w13_weight = layer.w13_weight
            w2_weight = layer.w2_weight
            from sglang.srt.layers.moe.moe_runner.deep_gemm import DeepGemmMoeQuantInfo

            # Only use_fp8=False when SGLANG_DEEPEP_BF16_DISPATCH is true,
            # otherwise use_fp8=True for FP8 dispatch path
            use_fp8 = not envs.SGLANG_DEEPEP_BF16_DISPATCH.get()
            quant_info = DeepGemmMoeQuantInfo(
                w13_weight=w13_weight,
                w2_weight=w2_weight,
                use_fp8=use_fp8,
            )
            return self.runner.run(dispatch_output, quant_info)
        elif self.use_flashinfer_cutlass:
            topk_output = dispatch_output.topk_output
            output = flashinfer_cutlass_fused_moe(
                input=x,
                token_selected_experts=topk_output.topk_ids,
                token_final_scales=topk_output.topk_weights,
                fc1_expert_weights=layer.w13_weight,
```
**EN:** This segment of `UnquantizedFusedMoEMethod.forward_cuda()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `UnquantizedFusedMoEMethod.forward_cuda()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 475-519: UnquantizedFusedMoEMethod.forward_cuda() (part 2/3)
```python
                fc2_expert_weights=layer.w2_weight,
                output_dtype=x.dtype,
                quant_scales=None,
                ep_size=layer.moe_ep_size,
                ep_rank=layer.moe_ep_rank,
                tp_size=layer.moe_tp_size,
                tp_rank=layer.moe_tp_rank,
                tune_max_num_tokens=next_power_of_2(x.shape[0]),
                activation_type=(
                    ActivationType.Relu2
                    if moe_runner_config.activation == "relu2"
                    else ActivationType.Swiglu
                ),
            )[0]
            return StandardCombineInput(hidden_states=output)
        elif self.use_flashinfer_trtllm_moe:
            from sglang.srt.layers.moe.moe_runner.flashinfer_trtllm import (
                FlashInferTrtllmBf16MoeQuantInfo,
            )

            quant_info = FlashInferTrtllmBf16MoeQuantInfo(
                gemm1_weights=layer.w13_weight,
                gemm2_weights=layer.w2_weight,
                global_num_experts=layer.num_experts,
                local_expert_offset=layer.moe_ep_rank * layer.num_local_experts,
            )
            return self.runner.run(dispatch_output, quant_info)
        else:
            if self._aiter_runner is not None:
                from sglang.srt.layers.moe.moe_runner.aiter import (
                    AiterMoeQuantInfo,
                )

                try:
                    quant_info = AiterMoeQuantInfo(
                        w13_weight=layer.w13_weight,
                        w2_weight=layer.w2_weight,
                        expert_mask=layer.dispatcher.expert_mask_gpu,
                    )
                    return self._aiter_runner.run(dispatch_output, quant_info)
                except RuntimeError as e:
                    # AITER CK fused_moe may not support all GEMM dimensions
                    # (e.g. Gemma4 MoE with 128 experts x 704 intermediate size)
                    logger.warning_once(
                        f"AITER CK fused_moe failed ({e}), "
```
**EN:** This segment of `UnquantizedFusedMoEMethod.forward_cuda()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `UnquantizedFusedMoEMethod.forward_cuda()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 520-529: UnquantizedFusedMoEMethod.forward_cuda() (part 3/3)
```python
                        "falling back to Triton MoE runner."
                    )

            quant_info = TritonMoeQuantInfo(
                w13_weight=layer.w13_weight,
                w2_weight=layer.w2_weight,
                b13=getattr(layer, "w13_weight_bias", None),
                b2=getattr(layer, "w2_weight_bias", None),
            )
            return self.runner.run(dispatch_output, quant_info)
```
**EN:** This segment of `UnquantizedFusedMoEMethod.forward_cuda()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `UnquantizedFusedMoEMethod.forward_cuda()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 531-579: UnquantizedFusedMoEMethod.forward_cpu()
```python
    def forward_cpu(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        moe_runner_config = self.moe_runner_config

        assert (
            moe_runner_config.activation == "silu"
        ), f"activation = {moe_runner_config.activation} is not supported."

        if use_intel_amx_backend(layer):
            from sglang.srt.layers.moe.topk import apply_topk_weights_cpu

            topk_weights, topk_ids, _ = topk_output
            x, topk_weights = apply_topk_weights_cpu(
                moe_runner_config.apply_router_weight_on_input, topk_weights, x
            )
            output = torch.ops.sgl_kernel.fused_experts_cpu(
                x,
                layer.w13_weight,
                layer.w2_weight,
                topk_weights,
                topk_ids,
                False,  # inplace # See [Note] inplace should be False in fused_experts.
                CPUQuantMethod.UNQUANT,
                None,  # w1_scale
                None,  # w2_scale
                None,  # w1_zp
                None,  # w2_zp
                None,  # block_size
                True,  # is_vnni
            )
            return StandardCombineInput(hidden_states=output)
        else:
            from sglang.srt.layers.moe.fused_moe_native import moe_forward_native

            output = moe_forward_native(
                layer,
                x,
                topk_output,
                moe_runner_config,
            )
            return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `UnquantizedFusedMoEMethod.forward_cpu()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.forward_cpu()`，用于实现量化栈中的可复用模块逻辑。

### Lines 581-587: UnquantizedFusedMoEMethod.get_triton_quant_info()
```python
    def get_triton_quant_info(self, layer: torch.nn.Module) -> TritonMoeQuantInfo:
        return TritonMoeQuantInfo(
            w13_weight=layer.w13_weight,
            w2_weight=layer.w2_weight,
            b13=getattr(layer, "w13_weight_bias", None),
            b2=getattr(layer, "w2_weight_bias", None),
        )
```
**EN:** This block defines `UnquantizedFusedMoEMethod.get_triton_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.get_triton_quant_info()`，用于为调用方获取或计算派生值。

### Lines 589-635: UnquantizedFusedMoEMethod.forward_xpu()
```python
    def forward_xpu(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        moe_runner_config = self.moe_runner_config
        assert moe_runner_config.activation in [
            "silu",
            "gelu",
        ], f"activation = {moe_runner_config.activation} is not supported."

        backend = self.runner.runner_backend
        if use_intel_xpu_backend():
            # sgl-kernel-xpu path
            from sgl_kernel import fused_experts

            topk_weights, topk_ids, _ = topk_output
            if moe_runner_config.apply_router_weight_on_input:
                x = x * topk_weights.to(x.dtype)
                topk_weights = torch.ones_like(topk_weights)
            output = fused_experts(
                x,
                layer.w13_weight,
                layer.w2_weight,
                topk_weights,
                topk_ids,
                b1=getattr(layer, "w13_weight_bias", None),
                b2=getattr(layer, "w2_weight_bias", None),
                activation=moe_runner_config.activation,
                gemm1_alpha=moe_runner_config.gemm1_alpha,
                gemm1_limit=moe_runner_config.gemm1_clamp_limit,
            )
            return StandardCombineInput(hidden_states=output)
        else:
            assert backend.is_triton()
            assert (
                moe_runner_config.activation == "silu"
            ), f"activation = {moe_runner_config.activation} is not supported \
            for Triton PATH, please set ENV SGLANG_USE_SGL_XPU=1."

            quant_info = self.get_triton_quant_info(layer)
            return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `UnquantizedFusedMoEMethod.forward_xpu()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.forward_xpu()`，用于实现量化栈中的可复用模块逻辑。

### Lines 637-681: UnquantizedFusedMoEMethod.forward_npu() (part 1/2)
```python
    def forward_npu(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:

        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        # x.shape = [B*S, H]
        x = dispatch_output.hidden_states
        # topk_weights.shape = [B*S, K]; topk_ids.shape = [B*S, K]
        topk_weights, topk_ids, _ = dispatch_output.topk_output

        original_dtype = x.dtype
        num_tokens = x.shape[0]
        topk_weights = topk_weights.to(x.dtype)
        topk_ids = topk_ids.to(torch.int32)
        num_experts = layer.num_experts
        top_k = layer.top_k or topk_ids.shape[1]  # in case layer.top_k is not set

        hidden_states, expanded_row_idx, expert_tokens, _ = (
            torch.ops.npu.npu_moe_init_routing_v2(
                x,
                topk_ids,
                active_num=num_tokens * top_k,
                expert_num=num_experts,
                expert_tokens_num_type=1,
                expert_tokens_num_flag=True,
                active_expert_range=[0, num_experts],
                quant_mode=-1,
            )
        )
        expert_tokens = expert_tokens.to(torch.int64)
        w13_bias = [layer.w13_weight_bias] if self.with_bias else None
        w2_bias = [layer.w2_weight_bias] if self.with_bias else None

        # gmm1: gate_up_proj
        hidden_states = torch.ops.npu.npu_grouped_matmul(
            x=[hidden_states],
            weight=[layer.w13_weight],
            bias=w13_bias,
            split_item=2,
            group_list_type=1,
            group_type=0,
            group_list=expert_tokens,
```
**EN:** This segment of `UnquantizedFusedMoEMethod.forward_npu()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `UnquantizedFusedMoEMethod.forward_npu()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 682-720: UnquantizedFusedMoEMethod.forward_npu() (part 2/2)
```python
            output_dtype=original_dtype,
        )[0]

        # act_fn:
        if self.moe_runner_config.activation == "npu_swiglu_oai":
            from sgl_kernel_npu.activation.swiglu_oai import swiglu_oai

            hidden_states = swiglu_oai(layer, hidden_states)
        elif self.moe_runner_config.activation == "silu":
            hidden_states = torch.ops.npu.npu_swiglu(hidden_states)
        else:
            from sglang.srt.layers.activation import GeluAndMul

            hidden_states = GeluAndMul()(hidden_states)

        # gmm2: down_proj
        hidden_states = torch.ops.npu.npu_grouped_matmul(
            x=[hidden_states],
            weight=[layer.w2_weight],
            bias=w2_bias,
            split_item=2,
            group_list_type=1,
            group_type=0,
            group_list=expert_tokens,
            output_dtype=original_dtype,
        )[0]

        final_hidden_states = torch.ops.npu.npu_moe_finalize_routing(
            hidden_states,
            skip1=None,
            skip2=None,
            bias=None,
            scales=topk_weights,
            expanded_src_to_dst_row=expanded_row_idx,
            export_for_source_row=topk_ids,
            drop_pad_mode=2,
        )

        return StandardCombineInput(hidden_states=final_hidden_states)
```
**EN:** This segment of `UnquantizedFusedMoEMethod.forward_npu()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `UnquantizedFusedMoEMethod.forward_npu()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 722-723: UnquantizedFusedMoEMethod.forward_tpu()
```python
    def forward_tpu(self, *args, **kwargs) -> CombineInput:
        raise NotImplementedError("The TPU backend currently does not support MoE.")
```
**EN:** This block defines `UnquantizedFusedMoEMethod.forward_tpu()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.forward_tpu()`，用于实现量化栈中的可复用模块逻辑。

### Lines 725-726: UnquantizedFusedMoEMethod.forward_musa()
```python
    def forward_musa(self, *args, **kwargs) -> CombineInput:
        return self.forward_cuda(*args, **kwargs)
```
**EN:** This block defines `UnquantizedFusedMoEMethod.forward_musa()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `UnquantizedFusedMoEMethod.forward_musa()`，用于实现量化栈中的可复用模块逻辑。

### Lines 728-728: UnquantizedFusedMoEMethod member: initialize forward_native
```python
    forward_native = forward_cpu
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as forward_native.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 forward_native。

## Key Concepts / 关键概念
- `UnquantizedEmbeddingMethod`: A runtime method class that structures file-level quantization behavior. / `UnquantizedEmbeddingMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `UnquantizedLinearMethod`: A runtime method class that structures file-level quantization behavior. / `UnquantizedLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `UnquantizedFusedMoEMethod`: A runtime method class that structures file-level quantization behavior. / `UnquantizedFusedMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `aiter.ops.shuffle`, `aiter.tuned_gemm`, `flashinfer.fused_moe`, `flashinfer.fused_moe.core`, `logging`, `sgl_kernel`, `sgl_kernel_npu.activation.swiglu_oai`, `torch`, `torch.nn.functional`, `torch.nn.parameter`, `typing`
- **Internal / 内部**: `sglang.srt.environ`, `sglang.srt.hardware_backend.npu.utils`, `sglang.srt.layers.activation`, `sglang.srt.layers.amx_utils`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_native`, `sglang.srt.layers.moe.moe_runner.aiter`, `sglang.srt.layers.moe.moe_runner.deep_gemm`, `sglang.srt.layers.moe.moe_runner.flashinfer_trtllm`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.moe_runner.triton_kernels`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.topk`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.utils`, `sglang.srt.utils`
