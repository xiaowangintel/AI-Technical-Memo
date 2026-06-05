# deep_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/deep_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the DeepGEMM scaled matrix-multiplication kernel path. / 实现 DeepGEMM 的缩放矩阵乘内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-29)
```python
import torch

import vllm.envs as envs
from vllm.config import get_current_vllm_config
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    deepgemm_post_process_fp8_weight_block,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
)
from vllm.model_executor.utils import replace_parameter
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import (
    fp8_gemm_nt,
    is_deep_gemm_e8m0_used,
    is_deep_gemm_supported,
    should_auto_disable_deep_gemm,
    should_use_deepgemm_for_fp8_linear,
)
from vllm.utils.torch_utils import direct_register_custom_op

from .BlockScaledMMLinearKernel import (
    Fp8BlockScaledMMLinearKernel,
    FP8ScaledMMLinearLayerConfig,
)
```
**EN:** This import block loads `torch`, `vllm.envs`, `vllm.config`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `deep_gemm.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm.envs`, `vllm.config`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, ...，为 `deep_gemm.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `DeepGemmFp8BlockScaledMMKernel` (lines 32-123)
```python
class DeepGemmFp8BlockScaledMMKernel(Fp8BlockScaledMMLinearKernel):
    def __init__(self, config: FP8ScaledMMLinearLayerConfig):
        super().__init__(config)
        self.use_deep_gemm_e8m0 = is_deep_gemm_e8m0_used()
        act_scale_descriptor = config.activation_quant_key.scale
        self.is_deep_gemm_supported = is_deep_gemm_supported()
        self.quant_fp8 = QuantFP8(
            static=False,
            group_shape=act_scale_descriptor.group_shape,
            use_ue8m0=self.use_deep_gemm_e8m0,
            tma_aligned_scales=envs.VLLM_USE_DEEP_GEMM_TMA_ALIGNED_SCALES,
            column_major_scales=True,
        )

    @classmethod
    def is_supported(cls, compute_capability=None):
        if not current_platform.is_cuda():
            return False, "DeepGEMM is only supported on cuda platform"
        if not is_deep_gemm_supported():
            return False, "Currently, only Hopper and Blackwell GPUs are supported."
        return True, None

    @classmethod
    def can_implement(cls, config):
        can_implement_base, reason = super().can_implement(config)
        if not can_implement_base:
            return can_implement_base, reason
        if config.out_dtype != torch.bfloat16:
            return (False, "Supports only output dtype of bfloat16")

        act_quant_desc = config.activation_quant_key.scale
        if act_quant_desc.group_shape != GroupShape(1, 128):
            return (
                False,
                "Supports only dynamic per token group activation "
                "quantization with group_shape=(1,128).",
            )
        model_config = get_current_vllm_config().model_config

        if model_config is None:
            return False, "Model configuration is required."

        model_type = getattr(model_config.hf_text_config, "model_type", None)
        if should_auto_disable_deep_gemm(model_type):
            return False, f"Should not use deepgemm for model {model_type}"

        if not should_use_deepgemm_for_fp8_linear(
            config.out_dtype, config.weight_shape
        ):
            return False, "The provided metadata is not supported."
        return True, None

    def process_weights_after_loading(self, layer):
        super().process_weights_after_loading(layer)
        params = self._get_layer_params(layer)
        assert layer.weight_block_size is not None

        if self.is_deep_gemm_supported:
            weight_scale_invs = params.weight_scale_inv
            scale_attr = (
                params.WEIGHT_SCALE_INV
                if weight_scale_invs is not None
                else params.WEIGHT_SCALE
            )
            dg_weight, dg_weight_scale = deepgemm_post_process_fp8_weight_block(
                wq=params.weight,
                ws=weight_scale_invs
                if weight_scale_invs is not None
                else params.weight_scale,
                quant_block_shape=tuple(layer.weight_block_size),
                use_e8m0=self.use_deep_gemm_e8m0,
                is_bmm=getattr(layer, "is_bmm", False),
                bmm_batch_size=getattr(layer, "bmm_batch_size", 0),
            )
            replace_parameter(layer, params.WEIGHT, dg_weight)
            replace_parameter(layer, scale_attr, dg_weight_scale)

    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        out_dtype = self.config.out_dtype
        output = torch.empty(
            (A.shape[0], B.shape[0]),
            dtype=out_dtype,
            device=A.device,
        )
        torch.ops.vllm.fp8_gemm_nt_op(A, As, B, Bs, output, self.use_deep_gemm_e8m0)
        return output
```
**EN:** This kernel class defines `DeepGemmFp8BlockScaledMMKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `Fp8BlockScaledMMLinearKernel`. Key methods include `__init__`, `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_block_scaled_mm`.
**CN:** 该内核类定义了 `DeepGemmFp8BlockScaledMMKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `Fp8BlockScaledMMLinearKernel`。 关键方法包括 `__init__`, `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_block_scaled_mm`。

### Method `DeepGemmFp8BlockScaledMMKernel.__init__` (lines 33-44)
```python
    def __init__(self, config: FP8ScaledMMLinearLayerConfig):
        super().__init__(config)
        self.use_deep_gemm_e8m0 = is_deep_gemm_e8m0_used()
        act_scale_descriptor = config.activation_quant_key.scale
        self.is_deep_gemm_supported = is_deep_gemm_supported()
        self.quant_fp8 = QuantFP8(
            static=False,
            group_shape=act_scale_descriptor.group_shape,
            use_ue8m0=self.use_deep_gemm_e8m0,
            tma_aligned_scales=envs.VLLM_USE_DEEP_GEMM_TMA_ALIGNED_SCALES,
            column_major_scales=True,
        )
```
**EN:** This method implements `DeepGemmFp8BlockScaledMMKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `__init__`, `is_deep_gemm_e8m0_used`, `is_deep_gemm_supported`, `QuantFP8`.
**CN:** 该方法 `DeepGemmFp8BlockScaledMMKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `__init__`, `is_deep_gemm_e8m0_used`, `is_deep_gemm_supported`, `QuantFP8` 等例程。

### Method `DeepGemmFp8BlockScaledMMKernel.is_supported` (lines 47-52)
```python
    def is_supported(cls, compute_capability=None):
        if not current_platform.is_cuda():
            return False, "DeepGEMM is only supported on cuda platform"
        if not is_deep_gemm_supported():
            return False, "Currently, only Hopper and Blackwell GPUs are supported."
        return True, None
```
**EN:** This method implements `DeepGemmFp8BlockScaledMMKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cuda`, `is_deep_gemm_supported`.
**CN:** 该方法 `DeepGemmFp8BlockScaledMMKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_cuda`, `is_deep_gemm_supported` 等例程。

### Method `DeepGemmFp8BlockScaledMMKernel.can_implement` (lines 55-82)
```python
    def can_implement(cls, config):
        can_implement_base, reason = super().can_implement(config)
        if not can_implement_base:
            return can_implement_base, reason
        if config.out_dtype != torch.bfloat16:
            return (False, "Supports only output dtype of bfloat16")

        act_quant_desc = config.activation_quant_key.scale
        if act_quant_desc.group_shape != GroupShape(1, 128):
            return (
                False,
                "Supports only dynamic per token group activation "
                "quantization with group_shape=(1,128).",
            )
        model_config = get_current_vllm_config().model_config

        if model_config is None:
            return False, "Model configuration is required."

        model_type = getattr(model_config.hf_text_config, "model_type", None)
        if should_auto_disable_deep_gemm(model_type):
            return False, f"Should not use deepgemm for model {model_type}"

        if not should_use_deepgemm_for_fp8_linear(
            config.out_dtype, config.weight_shape
        ):
            return False, "The provided metadata is not supported."
        return True, None
```
**EN:** This method implements `DeepGemmFp8BlockScaledMMKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `can_implement`, `getattr`, `should_auto_disable_deep_gemm`, `GroupShape`.
**CN:** 该方法 `DeepGemmFp8BlockScaledMMKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `can_implement`, `getattr`, `should_auto_disable_deep_gemm`, `GroupShape` 等例程。

### Method `DeepGemmFp8BlockScaledMMKernel.process_weights_after_loading` (lines 84-107)
```python
    def process_weights_after_loading(self, layer):
        super().process_weights_after_loading(layer)
        params = self._get_layer_params(layer)
        assert layer.weight_block_size is not None

        if self.is_deep_gemm_supported:
            weight_scale_invs = params.weight_scale_inv
            scale_attr = (
                params.WEIGHT_SCALE_INV
                if weight_scale_invs is not None
                else params.WEIGHT_SCALE
            )
            dg_weight, dg_weight_scale = deepgemm_post_process_fp8_weight_block(
                wq=params.weight,
                ws=weight_scale_invs
                if weight_scale_invs is not None
                else params.weight_scale,
                quant_block_shape=tuple(layer.weight_block_size),
                use_e8m0=self.use_deep_gemm_e8m0,
                is_bmm=getattr(layer, "is_bmm", False),
                bmm_batch_size=getattr(layer, "bmm_batch_size", 0),
            )
            replace_parameter(layer, params.WEIGHT, dg_weight)
            replace_parameter(layer, scale_attr, dg_weight_scale)
```
**EN:** This method implements `DeepGemmFp8BlockScaledMMKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `process_weights_after_loading`, `_get_layer_params`, `deepgemm_post_process_fp8_weight_block`, `replace_parameter`.
**CN:** 该方法 `DeepGemmFp8BlockScaledMMKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `process_weights_after_loading`, `_get_layer_params`, `deepgemm_post_process_fp8_weight_block`, `replace_parameter` 等例程。

### Method `DeepGemmFp8BlockScaledMMKernel.apply_block_scaled_mm` (lines 109-123)
```python
    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        out_dtype = self.config.out_dtype
        output = torch.empty(
            (A.shape[0], B.shape[0]),
            dtype=out_dtype,
            device=A.device,
        )
        torch.ops.vllm.fp8_gemm_nt_op(A, As, B, Bs, output, self.use_deep_gemm_e8m0)
        return output
```
**EN:** This method implements `DeepGemmFp8BlockScaledMMKernel.apply_block_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `empty`, `fp8_gemm_nt_op`.
**CN:** 该方法 `DeepGemmFp8BlockScaledMMKernel.apply_block_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply block scaled mm 相关工作。 它内部会调用 `empty`, `fp8_gemm_nt_op` 等例程。

### Function `_fp8_gemm_nt_op` (lines 126-139)
```python
def _fp8_gemm_nt_op(
    q_input: torch.Tensor,
    input_scale: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    output: torch.Tensor,
    use_deep_gemm_e8m0: bool,
) -> None:
    fp8_gemm_nt(
        (q_input, input_scale),
        (weight, weight_scale),
        output,
        is_deep_gemm_e8m0_used=use_deep_gemm_e8m0,
    )
```
**EN:** This helper implements `_fp8_gemm_nt_op`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `fp8_gemm_nt`.
**CN:** 该函数 `_fp8_gemm_nt_op` 封装了此模块中的一段关键运行时逻辑，重点处理 fp 8 gemm nt op 相关工作。 它内部会调用 `fp8_gemm_nt` 等例程。

### Function `_fp8_gemm_nt_op_fake` (lines 142-150)
```python
def _fp8_gemm_nt_op_fake(
    q_input: torch.Tensor,
    input_scale: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    output: torch.Tensor,
    use_deep_gemm_e8m0: bool,
) -> None:
    return None
```
**EN:** This helper implements `_fp8_gemm_nt_op_fake`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该函数 `_fp8_gemm_nt_op_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 fp 8 gemm nt op fake 相关工作。

### Top-level block (lines 153-158)
```python
direct_register_custom_op(
    "fp8_gemm_nt_op",
    _fp8_gemm_nt_op,
    mutates_args=["output"],
    fake_impl=_fp8_gemm_nt_op_fake,
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

## Key Concepts / 关键概念
- **FP8 quantization / FP8 量化**
  - **EN:** The module handles FP8 data, scales, or kernels for low-precision inference.
  - **CN:** 该模块处理 FP8 数据、缩放因子或相关低精度推理内核。
- **Scaled matrix multiplication / 缩放矩阵乘**
  - **EN:** The code centers on scaled matrix multiplication kernels and their dispatch rules.
  - **CN:** 代码围绕缩放矩阵乘内核及其分派规则展开。
- **Linear kernel abstraction / 线性内核抽象**
  - **EN:** The module defines abstractions or selectors for quantized linear operators.
  - **CN:** 该模块定义量化线性算子的抽象或选择器。
- **Kernel registration / 内核注册**
  - **EN:** The file registers operators, backends, or preset configurations with PyTorch/vLLM.
  - **CN:** 该文件向 PyTorch/vLLM 注册算子、后端或预设配置。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.envs`, `vllm.config`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`, `.BlockScaledMMLinearKernel`
- **External / 外部依赖**: `torch`
