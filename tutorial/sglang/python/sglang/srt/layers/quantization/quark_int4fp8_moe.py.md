# quark_int4fp8_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/quark_int4fp8_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for quark int4fp8 moe quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 Quark int4fp8 MoE 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module imports and setup
```python
import logging
from typing import TYPE_CHECKING, Any, Dict, List, Optional

import torch
from tqdm import tqdm
from tqdm.std import EMA

from sglang.srt.distributed import get_tensor_model_parallel_rank
from sglang.srt.layers.int4fp8_utils import (
    pack_int4_to_int32,
    quantize_fp8_scale_tensorwise,
    quantize_int4_scale_columnwise,
)
from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.fp8 import Fp8LinearMethod
from sglang.srt.utils import BAR_FORMAT, is_hip, set_weight_attrs
```
**EN:** This block imports aiter.ops.shuffle, logging, torch, tqdm, sglang.srt.distributed, sglang.srt.layers.int4fp8_utils, sglang.srt.layers.linear, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 aiter.ops.shuffle, logging, torch, tqdm, sglang.srt.distributed, sglang.srt.layers.int4fp8_utils, sglang.srt.layers.linear, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 23-24: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import DispatchOutput
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 26-26: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 29-32: conditional logic for _is_hip
```python
if _is_hip:
    from aiter.ops.shuffle import shuffle_weight

    ON_GFX950 = "gfx950" in torch.cuda.get_device_properties("cuda").gcnArchName
```
**EN:** This block applies conditional logic controlled by `_is_hip`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_hip` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 34-34: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 37-47: tqdm_reset_no_print()
```python
def tqdm_reset_no_print(tqdm_bar: tqdm, total=None):
    tqdm_bar.n = 0
    if total is not None:
        tqdm_bar.total = total
    if tqdm_bar.disable:
        return
    tqdm_bar.last_print_n = 0
    tqdm_bar.last_print_t = tqdm_bar.start_t = tqdm_bar._time()
    tqdm_bar._ema_dn = EMA(tqdm_bar.smoothing)
    tqdm_bar._ema_dt = EMA(tqdm_bar.smoothing)
    tqdm_bar._ema_miniters = EMA(tqdm_bar.smoothing)
```
**EN:** This block defines `tqdm_reset_no_print()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `tqdm_reset_no_print()`，用于实现量化栈中的可复用模块逻辑。

### Lines 50-56: class QuarkInt4Fp8Config: definition
```python
class QuarkInt4Fp8Config(QuantizationConfig):
    """Config class for Quark Quantization.

    - Weight: static, per-channel, symmetric
    - Activation: dynamic, per-token, symmetric
    """
```
**EN:** This block declares `QuarkInt4Fp8Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_supported_act_dtypes, get_min_capability, get_name.
**CN:** 该代码块声明 `QuarkInt4Fp8Config`，它是量化栈中的配置类，组织了 __init__, get_supported_act_dtypes, get_min_capability, get_name 等行为。

### Lines 57-84: QuarkInt4Fp8Config.__init__()
```python
    def __init__(
        self,
        is_checkpoint_fp8_serialized: bool = False,
        activation_scheme: str = "dynamic",
    ):
        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized
        self.activation_scheme = activation_scheme

        if activation_scheme != "dynamic":
            raise NotImplementedError(
                "QuarkInt4Fp8Config only supports activation_scheme='dynamic'."
            )

        self.weight_block_size = None

        self.num_quant_layers = 0

        tp_rank = get_tensor_model_parallel_rank()

        # The weight iterator already has a progress bar on rank=0, account for that.
        position = 1 + tqdm._get_free_pos()
        self.online_quant_progress_bar = tqdm(
            total=0,
            desc=f"Online quark_int4fp8_moe quantization on rank={tp_rank}",
            position=position,
            bar_format=BAR_FORMAT,
            mininterval=2.0,
        )
```
**EN:** This block defines `QuarkInt4Fp8Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkInt4Fp8Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 86-88: QuarkInt4Fp8Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.float16, torch.bfloat16]
```
**EN:** This block defines `QuarkInt4Fp8Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkInt4Fp8Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 90-92: QuarkInt4Fp8Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** This block defines `QuarkInt4Fp8Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkInt4Fp8Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 94-96: QuarkInt4Fp8Config.get_name()
```python
    @classmethod
    def get_name(self) -> str:
        return "quark_int4fp8_moe"
```
**EN:** This block defines `QuarkInt4Fp8Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkInt4Fp8Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 98-100: QuarkInt4Fp8Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return []
```
**EN:** This block defines `QuarkInt4Fp8Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkInt4Fp8Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 102-104: QuarkInt4Fp8Config.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> "QuarkInt4Fp8Config":
        return cls()
```
**EN:** This block defines `QuarkInt4Fp8Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `QuarkInt4Fp8Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 106-121: QuarkInt4Fp8Config.get_quant_method()
```python
    def get_quant_method(
        self,
        layer: torch.nn.Module,
        prefix: str,
    ) -> Optional["QuantizeMethodBase"]:
        # TODO: fix circular imports issues in sglang forcing us to import here instead of at
        # the top of file.
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if isinstance(layer, LinearBase):
            return Fp8LinearMethod(self)
        elif isinstance(layer, FusedMoE):
            return QuarkInt4Fp8MoEMethod(self)

        return None
```
**EN:** This block defines `QuarkInt4Fp8Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkInt4Fp8Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 123-124: QuarkInt4Fp8Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `QuarkInt4Fp8Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkInt4Fp8Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 127-135: class QuarkInt4Fp8MoEMethod: definition
```python
class QuarkInt4Fp8MoEMethod(FusedMoEMethodBase):
    """MoE method for INT4FP8.

    Supports loading BF16/FP16 checkpoints, quantizing down to INT4, and dequantizing to FP8 during inference.

    Args:
        quant_config: The quantization config.
    """
```
**EN:** This block declares `QuarkInt4Fp8MoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, get_weight_loader, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `QuarkInt4Fp8MoEMethod`，它是量化栈中的运行方法类，组织了 __init__, get_weight_loader, create_weights, process_weights_after_loading 等行为。

### Lines 136-146: QuarkInt4Fp8MoEMethod.__init__()
```python
    def __init__(self, quant_config):
        self.quant_config = quant_config

        self.online_quant_progress_bar = self.quant_config.online_quant_progress_bar

        self.tp_rank = get_tensor_model_parallel_rank()

        if not _is_hip:
            raise NotImplementedError(
                "The quark_int4fp8_moe online quantization scheme is only supported on AMD GPUs."
            )
```
**EN:** This block defines `QuarkInt4Fp8MoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkInt4Fp8MoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 148-192: QuarkInt4Fp8MoEMethod.get_weight_loader() (part 1/3)
```python
    def get_weight_loader(self, layer, original_weight_loader):
        def online_int4_fp8_weight_loader(
            param: torch.nn.Parameter,
            loaded_weight: torch.Tensor,
            weight_name: str,
            shard_id: str,
            expert_id: int,
        ):
            if shard_id in ["w1", "w3"]:
                shard_size = self.w13_shard_size
            else:
                shard_size = self.w2_shard_size

            original_use_presharded_weights = layer.use_presharded_weights

            if not layer.use_presharded_weights:
                # In case the model is not pre-sharded (most checkpoints on HF Hub),
                # we shard the model here in order to run online quantization on
                # already sharded weights.
                # Some models as `lmzheng/grok-1` are already be sharded.
                layer.use_presharded_weights = True

                if shard_id in ["w1", "w3"]:
                    shard_dim = 0
                    loaded_weight = loaded_weight.narrow(
                        shard_dim, shard_size * self.tp_rank, shard_size
                    )
                else:
                    shard_dim = 1
                    loaded_weight = loaded_weight.narrow(
                        shard_dim, shard_size * self.tp_rank, shard_size
                    )

            # We want to run online quantization on-device for speed purposes.
            loaded_weight = loaded_weight.to(param.device)

            _, fp8_scale = quantize_fp8_scale_tensorwise(loaded_weight)

            int4_w, int4_scale = quantize_int4_scale_columnwise(loaded_weight)

            int4_w = pack_int4_to_int32(int4_w)
            int4_scale /= fp8_scale

            if shard_id in ["w1", "w3"]:
                if shard_id == "w1":
```
**EN:** This segment of `QuarkInt4Fp8MoEMethod.get_weight_loader()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `QuarkInt4Fp8MoEMethod.get_weight_loader()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 193-237: QuarkInt4Fp8MoEMethod.get_weight_loader() (part 2/3)
```python
                    shard_slice = slice(0, shard_size)
                    idx = 0
                else:
                    shard_slice = slice(shard_size, 2 * shard_size)
                    idx = 1

                assert param[expert_id][shard_slice].dtype == int4_w.dtype

                assert (
                    layer.w13_int4_scale[expert_id][shard_slice].shape
                    == int4_scale.shape
                )
                assert (
                    layer.w13_int4_scale[expert_id][shard_slice].dtype
                    == int4_scale.dtype
                )

                layer.w13_int4_scale[expert_id][shard_slice].copy_(int4_scale)

                assert layer.w13_fp8_scale[expert_id][idx].shape == fp8_scale.shape
                assert layer.w13_fp8_scale[expert_id][idx].dtype == fp8_scale.dtype

                layer.w13_fp8_scale[expert_id][idx].copy_(fp8_scale)
            else:
                assert param[expert_id].dtype == int4_w.dtype
                assert param[expert_id].shape == int4_w.shape

                assert layer.w2_int4_scale[expert_id].shape == int4_scale.shape
                assert layer.w2_int4_scale[expert_id].dtype == int4_scale.dtype

                layer.w2_int4_scale[expert_id].copy_(int4_scale)

                assert layer.w2_fp8_scale[expert_id].shape == fp8_scale.shape
                assert layer.w2_fp8_scale[expert_id].dtype == fp8_scale.dtype

                layer.w2_fp8_scale[expert_id].copy_(fp8_scale)

            original_weight_loader(
                param,
                int4_w,
                shard_id=shard_id,
                weight_name=weight_name,
                expert_id=expert_id,
            )
```
**EN:** This segment of `QuarkInt4Fp8MoEMethod.get_weight_loader()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `QuarkInt4Fp8MoEMethod.get_weight_loader()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 238-243: QuarkInt4Fp8MoEMethod.get_weight_loader() (part 3/3)
```python
            # Reset `use_presharded_weights` as the same layer may load several different weights.
            layer.use_presharded_weights = original_use_presharded_weights

            self.online_quant_progress_bar.update(1)

        return online_int4_fp8_weight_loader
```
**EN:** This segment of `QuarkInt4Fp8MoEMethod.get_weight_loader()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `QuarkInt4Fp8MoEMethod.get_weight_loader()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 245-289: QuarkInt4Fp8MoEMethod.create_weights() (part 1/3)
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
        # TODO: fix circular imports issues in sglang forcing us to import here instead of at
        # the top of file.
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        # print("intermediate_size_per_partition", intermediate_size_per_partition)
        # fused moe logic already hands TP logic.
        self.w13_shard_size = intermediate_size_per_partition
        self.w2_shard_size = intermediate_size_per_partition

        assert "weight_loader" in extra_weight_attrs
        original_weight_loader = extra_weight_attrs.get("weight_loader")

        online_int4fp8_weight_loader = self.get_weight_loader(
            layer, original_weight_loader
        )
        extra_weight_attrs["weight_loader"] = online_int4fp8_weight_loader

        params_dtype = torch.uint32
        # WEIGHTS
        # INT4 MoE weight - INT32 packed
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // 8,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // 8,
                dtype=params_dtype,
            ),
```
**EN:** This segment of `QuarkInt4Fp8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QuarkInt4Fp8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 290-334: QuarkInt4Fp8MoEMethod.create_weights() (part 2/3)
```python
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # Allocate 2 scales for w1 and w3 respectively.
        # They will be combined to a single scale after weight loading.
        w13_fp8_scale = torch.nn.Parameter(
            torch.ones(num_experts, 2, dtype=torch.float32), requires_grad=False
        )
        w2_fp8_scale = torch.nn.Parameter(
            torch.ones(num_experts, dtype=torch.float32), requires_grad=False
        )
        layer.register_parameter("w13_fp8_scale", w13_fp8_scale)
        layer.register_parameter("w2_fp8_scale", w2_fp8_scale)

        if _is_hip:
            w13_int4_scale = torch.nn.Parameter(
                torch.ones(
                    num_experts,
                    2 * intermediate_size_per_partition,
                    dtype=torch.float32,
                ),
                requires_grad=False,
            )
            w2_int4_scale = torch.nn.Parameter(
                torch.ones(num_experts, hidden_size, dtype=torch.float32),
                requires_grad=False,
            )
            layer.register_parameter("w13_int4_scale", w13_int4_scale)
            layer.register_parameter("w2_int4_scale", w2_int4_scale)

        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.TENSOR.value}
        )

        set_weight_attrs(w13_fp8_scale, extra_weight_attrs)
        set_weight_attrs(w2_fp8_scale, extra_weight_attrs)

        # Add the quantization method used (per tensor/grouped/channel)
        # to ensure the weight scales are loaded in properly
        extra_weight_attrs.update(
```
**EN:** This segment of `QuarkInt4Fp8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QuarkInt4Fp8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 335-349: QuarkInt4Fp8MoEMethod.create_weights() (part 3/3)
```python
            {"quant_method": FusedMoeWeightScaleSupported.CHANNEL.value}
        )

        set_weight_attrs(w13_int4_scale, extra_weight_attrs)
        set_weight_attrs(w2_int4_scale, extra_weight_attrs)

        w13_input_scale = None
        layer.register_parameter("w13_input_scale", w13_input_scale)

        w2_input_scale = None
        layer.register_parameter("w2_input_scale", w2_input_scale)

        # Loading from the checkpoint w1, w2, w3 times the number of experts.
        total = self.online_quant_progress_bar.total + num_experts * 3
        tqdm_reset_no_print(self.online_quant_progress_bar, total=total)
```
**EN:** This segment of `QuarkInt4Fp8MoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `QuarkInt4Fp8MoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 351-401: QuarkInt4Fp8MoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if _is_hip and not ON_GFX950:
            # CDNA3 does not support OCP FP8E4M3FN, but uses FP8E4M3FNUZ.
            # CDNA4 supports OCP FP8E4M3FN.
            layer.w13_int4_scale *= 0.5
            layer.w2_int4_scale *= 0.5

            layer.w13_fp8_scale *= 2.0
            layer.w2_fp8_scale *= 2.0

        # TODO: and use_aiter_moe: add after triton kernel added
        # INT4-FP8 (INT4 MoE Weight, FP8 Compute)
        # Weight Permutation
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

        # INT4-FP8 : offset INT4 w13_int4_scale to single w13_fp8_scale
        # Fp8 moe kernel needs single fp8 w13_fp8_scale for w13 per expert.
        # We won't do requant each expert's fp8 weight (not direct available),
        # instead we adjust half of INT4 w13_int4_scale numbers
        assert layer.w13_fp8_scale is not None
        shard_size = layer.intermediate_size_per_partition
        max_w13_scales = layer.w13_fp8_scale.max(dim=1).values
        for expert_id in range(layer.num_experts):
            start = 0
            max_w13_scale_fp8 = max_w13_scales[expert_id]
            for shard_id in range(2):
                if layer.w13_fp8_scale[expert_id][shard_id] != max_w13_scale_fp8:
                    int4_rescale = (
                        layer.w13_fp8_scale[expert_id][shard_id] / max_w13_scale_fp8
                    )
                    layer.w13_int4_scale[expert_id][
                        start : start + shard_size
                    ] *= int4_rescale
                start += shard_size

        layer.w13_fp8_scale = torch.nn.Parameter(max_w13_scales, requires_grad=False)

        # special hack to asm_moe, which takes (weight_int4_scale * weight_scale) as post GEMM scaling
        # optimal design - shall apply per-column weight_int4_scale before GEMM, and weight_scale post
        for expert_id in range(layer.num_experts):
            layer.w13_int4_scale[expert_id] *= max_w13_scales[expert_id]
            layer.w2_int4_scale[expert_id] *= layer.w2_fp8_scale[expert_id]
```
**EN:** This block defines `QuarkInt4Fp8MoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `QuarkInt4Fp8MoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 403-420: QuarkInt4Fp8MoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        from sglang.srt.layers.moe.utils import (
            get_moe_a2a_backend,
            get_moe_runner_backend,
        )

        self.moe_runner_config = moe_runner_config
        moe_runner_backend = get_moe_runner_backend()
        if moe_runner_backend.is_auto() and get_moe_a2a_backend().supports_aiter():
            moe_runner_backend = MoeRunnerBackend.AITER

        if moe_runner_backend.is_aiter():
            self.runner = MoeRunner(moe_runner_backend, moe_runner_config)
        else:
            # TODO(cwan): refactor other backends
            pass
```
**EN:** This block defines `QuarkInt4Fp8MoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuarkInt4Fp8MoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 422-446: QuarkInt4Fp8MoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: "DispatchOutput",
    ) -> torch.Tensor:
        from sglang.srt.layers.moe.moe_runner.aiter import (
            AiterMoeQuantInfo,
            AiterQuantType,
        )

        moe_runner_config = self.moe_runner_config

        # TODO: add triton kernel and add check get_bool_env_var("CK_MOE")
        assert (
            not moe_runner_config.no_combine
        ), f"no_combine={moe_runner_config.no_combine} is not supported."

        quant_info = AiterMoeQuantInfo(
            w13_weight=layer.w13_weight,
            w2_weight=layer.w2_weight,
            quant_type=AiterQuantType.PER_TOKEN,
            w13_scale=layer.w13_int4_scale,
            w2_scale=layer.w2_int4_scale,
        )
        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `QuarkInt4Fp8MoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuarkInt4Fp8MoEMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `tqdm_reset_no_print()` : A public function that implements reusable module logic for the quantization stack. / `tqdm_reset_no_print()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `QuarkInt4Fp8Config`: A configuration class that structures file-level quantization behavior. / `QuarkInt4Fp8Config` 是一个配置类，用于组织该文件中的量化行为。
- `QuarkInt4Fp8MoEMethod`: A runtime method class that structures file-level quantization behavior. / `QuarkInt4Fp8MoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `aiter.ops.shuffle`, `logging`, `torch`, `tqdm`, `tqdm.std`, `typing`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.layers.int4fp8_utils`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.aiter`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.moe.utils`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.fp8`, `sglang.srt.utils`
