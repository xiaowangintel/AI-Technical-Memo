# flashinfer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/flashinfer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the FlashInfer scaled matrix-multiplication kernel path. / 实现 FlashInfer 的缩放矩阵乘内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-34)
```python
from typing import ClassVar

import torch

import vllm.envs as envs
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    per_token_group_quant_fp8,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
)
from vllm.platforms import current_platform
from vllm.utils.flashinfer import (
    flashinfer_fp8_blockscale_gemm,
    flashinfer_scaled_fp8_mm,
    has_flashinfer,
    is_flashinfer_fp8_blockscale_gemm_supported,
    should_use_flashinfer_for_blockscale_fp8_gemm,
)
from vllm.utils.torch_utils import direct_register_custom_op

from .BlockScaledMMLinearKernel import (
    Fp8BlockScaledDynamicMMLinearKernel,
    Fp8BlockScaledMMLinearKernel,
)
from .deep_gemm import DeepGemmFp8BlockScaledMMKernel, fp8_gemm_nt
from .ScaledMMLinearKernel import (
    FP8ScaledMMLinearKernel,
    FP8ScaledMMLinearLayerConfig,
)
```
**EN:** This import block loads `typing`, `torch`, `vllm.envs`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `flashinfer.py`.
**CN:** 该导入代码块加载了 `typing`, `torch`, `vllm.envs`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, ...，为 `flashinfer.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Class `FlashInferFP8ScaledMMLinearKernel` (lines 37-78)
```python
class FlashInferFP8ScaledMMLinearKernel(FP8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cuda():
            return False, "requires CUDA."

        if not has_flashinfer():
            return False, "requires FlashInfer to be installed."

        if compute_capability is not None and compute_capability < 100:
            return False, "requires compute capability 100 and above."

        return True, None

    @classmethod
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        per_tensor_activation_scales = (
            c.activation_quant_key.scale.group_shape.is_per_tensor()
        )
        per_tensor_weight_scales = c.weight_quant_key.scale.group_shape.is_per_tensor()

        if not (per_tensor_activation_scales and per_tensor_weight_scales):
            return False, "requires per tensor activation and weight scales."

        return True, None

    def apply_scaled_mm(
        self,
        *,
        A: torch.Tensor,
        B: torch.Tensor,
        out_dtype: torch.dtype,
        As: torch.Tensor,
        Bs: torch.Tensor,
        bias: torch.Tensor | None,
        output_shape: list,
    ) -> torch.Tensor:
        return flashinfer_scaled_fp8_mm(
            A, B, out_dtype=out_dtype, scale_a=As, scale_b=Bs, bias=bias
        )
```
**EN:** This kernel class defines `FlashInferFP8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `FP8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `apply_scaled_mm`.
**CN:** 该内核类定义了 `FlashInferFP8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `FP8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `apply_scaled_mm`。

### Method `FlashInferFP8ScaledMMLinearKernel.is_supported` (lines 39-51)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_cuda():
            return False, "requires CUDA."

        if not has_flashinfer():
            return False, "requires FlashInfer to be installed."

        if compute_capability is not None and compute_capability < 100:
            return False, "requires compute capability 100 and above."

        return True, None
```
**EN:** This method implements `FlashInferFP8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cuda`, `has_flashinfer`.
**CN:** 该方法 `FlashInferFP8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_cuda`, `has_flashinfer` 等例程。

### Method `FlashInferFP8ScaledMMLinearKernel.can_implement` (lines 54-63)
```python
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        per_tensor_activation_scales = (
            c.activation_quant_key.scale.group_shape.is_per_tensor()
        )
        per_tensor_weight_scales = c.weight_quant_key.scale.group_shape.is_per_tensor()

        if not (per_tensor_activation_scales and per_tensor_weight_scales):
            return False, "requires per tensor activation and weight scales."

        return True, None
```
**EN:** This method implements `FlashInferFP8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_per_tensor`.
**CN:** 该方法 `FlashInferFP8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `is_per_tensor` 等例程。

### Method `FlashInferFP8ScaledMMLinearKernel.apply_scaled_mm` (lines 65-78)
```python
    def apply_scaled_mm(
        self,
        *,
        A: torch.Tensor,
        B: torch.Tensor,
        out_dtype: torch.dtype,
        As: torch.Tensor,
        Bs: torch.Tensor,
        bias: torch.Tensor | None,
        output_shape: list,
    ) -> torch.Tensor:
        return flashinfer_scaled_fp8_mm(
            A, B, out_dtype=out_dtype, scale_a=As, scale_b=Bs, bias=bias
        )
```
**EN:** This method implements `FlashInferFP8ScaledMMLinearKernel.apply_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `flashinfer_scaled_fp8_mm`.
**CN:** 该方法 `FlashInferFP8ScaledMMLinearKernel.apply_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply scaled mm 相关工作。 它内部会调用 `flashinfer_scaled_fp8_mm` 等例程。

### Class `FlashInferFp8BlockScaledMMKernel` (lines 81-139)
```python
class FlashInferFp8BlockScaledMMKernel(Fp8BlockScaledMMLinearKernel):
    # FlashInfer accepts BF16 input and handles FP8 conversion internally.
    apply_input_quant: ClassVar[bool] = False

    def __init__(self, config: FP8ScaledMMLinearLayerConfig) -> None:
        super().__init__(config)

    @classmethod
    def can_implement(cls, config: FP8ScaledMMLinearLayerConfig):
        can_implement_base, reason = super().can_implement(config)
        if not can_implement_base:
            return can_implement_base, reason

        act_quant_desc = config.activation_quant_key.scale
        if act_quant_desc.group_shape != GroupShape(1, 128):
            return (
                False,
                "Supports only dynamic per token group activation "
                "quantization with group_shape=(1,128).",
            )

        if not should_use_flashinfer_for_blockscale_fp8_gemm(
            is_flashinfer_fp8_blockscale_gemm_supported(),
            config.out_dtype,
            config.input_dtype,
            config.weight_quant_key.dtype,
            config.weight_shape,
        ):
            return (
                False,
                "The provided metadata is not supported.",
            )

        return True, None

    @classmethod
    def is_supported(cls, compute_capability=None):
        if not current_platform.is_cuda():
            return False, "only cuda devices are supported."

        if not is_flashinfer_fp8_blockscale_gemm_supported():
            return False, "FlashInfer block-scale FP8 GEMM is not available."

        return True, None

    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        # A is BF16 — FlashInfer handles FP8 conversion internally.
        # As is a placeholder (apply_input_quant=False) and is not used here.
        return torch.ops.vllm.flashinfer_fp8_blockscale_gemm(
            A,  # BF16 input
            B,  # FP8 weight
            Bs,  # Weight scales
        )
```
**EN:** This kernel class defines `FlashInferFp8BlockScaledMMKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `Fp8BlockScaledMMLinearKernel`. Key methods include `__init__`, `can_implement`, `is_supported`, `apply_block_scaled_mm`.
**CN:** 该内核类定义了 `FlashInferFp8BlockScaledMMKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `Fp8BlockScaledMMLinearKernel`。 关键方法包括 `__init__`, `can_implement`, `is_supported`, `apply_block_scaled_mm`。

### Method `FlashInferFp8BlockScaledMMKernel.__init__` (lines 85-86)
```python
    def __init__(self, config: FP8ScaledMMLinearLayerConfig) -> None:
        super().__init__(config)
```
**EN:** This method implements `FlashInferFp8BlockScaledMMKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `__init__`, `super`.
**CN:** 该方法 `FlashInferFp8BlockScaledMMKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `__init__`, `super` 等例程。

### Method `FlashInferFp8BlockScaledMMKernel.can_implement` (lines 89-114)
```python
    def can_implement(cls, config: FP8ScaledMMLinearLayerConfig):
        can_implement_base, reason = super().can_implement(config)
        if not can_implement_base:
            return can_implement_base, reason

        act_quant_desc = config.activation_quant_key.scale
        if act_quant_desc.group_shape != GroupShape(1, 128):
            return (
                False,
                "Supports only dynamic per token group activation "
                "quantization with group_shape=(1,128).",
            )

        if not should_use_flashinfer_for_blockscale_fp8_gemm(
            is_flashinfer_fp8_blockscale_gemm_supported(),
            config.out_dtype,
            config.input_dtype,
            config.weight_quant_key.dtype,
            config.weight_shape,
        ):
            return (
                False,
                "The provided metadata is not supported.",
            )

        return True, None
```
**EN:** This method implements `FlashInferFp8BlockScaledMMKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `can_implement`, `GroupShape`, `should_use_flashinfer_for_blockscale_fp8_gemm`, `super`.
**CN:** 该方法 `FlashInferFp8BlockScaledMMKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `can_implement`, `GroupShape`, `should_use_flashinfer_for_blockscale_fp8_gemm`, `super` 等例程。

### Method `FlashInferFp8BlockScaledMMKernel.is_supported` (lines 117-124)
```python
    def is_supported(cls, compute_capability=None):
        if not current_platform.is_cuda():
            return False, "only cuda devices are supported."

        if not is_flashinfer_fp8_blockscale_gemm_supported():
            return False, "FlashInfer block-scale FP8 GEMM is not available."

        return True, None
```
**EN:** This method implements `FlashInferFp8BlockScaledMMKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_cuda`, `is_flashinfer_fp8_blockscale_gemm_supported`.
**CN:** 该方法 `FlashInferFp8BlockScaledMMKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_cuda`, `is_flashinfer_fp8_blockscale_gemm_supported` 等例程。

### Method `FlashInferFp8BlockScaledMMKernel.apply_block_scaled_mm` (lines 126-139)
```python
    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        # A is BF16 — FlashInfer handles FP8 conversion internally.
        # As is a placeholder (apply_input_quant=False) and is not used here.
        return torch.ops.vllm.flashinfer_fp8_blockscale_gemm(
            A,  # BF16 input
            B,  # FP8 weight
            Bs,  # Weight scales
        )
```
**EN:** This method implements `FlashInferFp8BlockScaledMMKernel.apply_block_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `flashinfer_fp8_blockscale_gemm`.
**CN:** 该方法 `FlashInferFp8BlockScaledMMKernel.apply_block_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply block scaled mm 相关工作。 它内部会调用 `flashinfer_fp8_blockscale_gemm` 等例程。

### Class `FlashInferFp8DeepGEMMDynamicBlockScaledKernel` (lines 142-189)
```python
class FlashInferFp8DeepGEMMDynamicBlockScaledKernel(
    Fp8BlockScaledDynamicMMLinearKernel
):
    """
    Conditional FlashInfer / DeepGEMM FP8 block-scaled GEMM.

    Dispatches between two kernels based on input batch size:
    - Small batches (M < 32): FlashInfer's swapAB trick for better utilisation.
    - Large batches (M >= 32): DeepGEMM for peak throughput.

    apply_input_quant is False because FlashInfer accepts BF16 input and
    handles FP8 conversion internally.  The DeepGEMM branch therefore
    quantises BF16→FP8 inside apply_mm via a closure before dispatching to
    the DeepGEMM kernel — keeping both branches compatible with the single
    BF16 tensor operand list passed by torch.cond.
    """

    base_type: ClassVar[type[FlashInferFp8BlockScaledMMKernel]] = (
        FlashInferFp8BlockScaledMMKernel
    )
    fallback_type: ClassVar[type[DeepGemmFp8BlockScaledMMKernel]] = (
        DeepGemmFp8BlockScaledMMKernel
    )
    apply_input_quant: ClassVar[bool] = False

    def __init__(self, config: FP8ScaledMMLinearLayerConfig):
        super().__init__(config)
        self.base: FlashInferFp8BlockScaledMMKernel
        self.fallback: DeepGemmFp8BlockScaledMMKernel

    def process_weights_after_loading(self, layer: torch.nn.Module):
        # DeepGEMM need post-processing; both kernels share the same
        # parameter tensor layout so processing once is sufficient.
        self.fallback.process_weights_after_loading(layer)

    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        group_size = self.weight_group_shape.col
        use_deep_gemm_e8m0 = self.fallback.use_deep_gemm_e8m0

        return torch.ops.vllm.dynamic_flashinfer_deepgemm_blockscale_gemm(
            A, B, Bs, group_size, use_deep_gemm_e8m0
        )
```
**EN:** This kernel class defines `FlashInferFp8DeepGEMMDynamicBlockScaledKernel`. Conditional FlashInfer / DeepGEMM FP8 block-scaled GEMM. It inherits from `Fp8BlockScaledDynamicMMLinearKernel`. Key methods include `__init__`, `process_weights_after_loading`, `apply_block_scaled_mm`.
**CN:** 该内核类定义了 `FlashInferFp8DeepGEMMDynamicBlockScaledKernel`。 它主要负责与 `FlashInferFp8DeepGEMMDynamicBlockScaledKernel` 对应的数据组织、接口约束或执行流程。 它继承自 `Fp8BlockScaledDynamicMMLinearKernel`。 关键方法包括 `__init__`, `process_weights_after_loading`, `apply_block_scaled_mm`。

### Method `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.__init__` (lines 167-170)
```python
    def __init__(self, config: FP8ScaledMMLinearLayerConfig):
        super().__init__(config)
        self.base: FlashInferFp8BlockScaledMMKernel
        self.fallback: DeepGemmFp8BlockScaledMMKernel
```
**EN:** This method implements `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `__init__`, `super`.
**CN:** 该方法 `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `__init__`, `super` 等例程。

### Method `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.process_weights_after_loading` (lines 172-175)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        # DeepGEMM need post-processing; both kernels share the same
        # parameter tensor layout so processing once is sufficient.
        self.fallback.process_weights_after_loading(layer)
```
**EN:** This method implements `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `process_weights_after_loading`.
**CN:** 该方法 `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `process_weights_after_loading` 等例程。

### Method `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.apply_block_scaled_mm` (lines 177-189)
```python
    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        group_size = self.weight_group_shape.col
        use_deep_gemm_e8m0 = self.fallback.use_deep_gemm_e8m0

        return torch.ops.vllm.dynamic_flashinfer_deepgemm_blockscale_gemm(
            A, B, Bs, group_size, use_deep_gemm_e8m0
        )
```
**EN:** This method implements `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.apply_block_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `dynamic_flashinfer_deepgemm_blockscale_gemm`.
**CN:** 该方法 `FlashInferFp8DeepGEMMDynamicBlockScaledKernel.apply_block_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply block scaled mm 相关工作。 它内部会调用 `dynamic_flashinfer_deepgemm_blockscale_gemm` 等例程。

### Function `_flashinfer_fp8_blockscale_gemm_impl` (lines 192-202)
```python
def _flashinfer_fp8_blockscale_gemm_impl(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
) -> torch.Tensor:
    return flashinfer_fp8_blockscale_gemm(
        input=input,
        weight=weight,
        weight_scale=weight_scale,
        out_dtype=torch.bfloat16,
    )
```
**EN:** This helper implements `_flashinfer_fp8_blockscale_gemm_impl`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `flashinfer_fp8_blockscale_gemm`.
**CN:** 该函数 `_flashinfer_fp8_blockscale_gemm_impl` 封装了此模块中的一段关键运行时逻辑，重点处理 flashinfer fp 8 blockscale gemm impl 相关工作。 它内部会调用 `flashinfer_fp8_blockscale_gemm` 等例程。

### Function `_flashinfer_fp8_blockscale_gemm_fake` (lines 205-215)
```python
def _flashinfer_fp8_blockscale_gemm_fake(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
) -> torch.Tensor:
    """
    Required fake/meta implementation for torch.compile graph tracing.
    """
    return torch.empty(
        input.shape[0], weight.shape[0], dtype=torch.bfloat16, device=input.device
    )
```
**EN:** This helper implements `_flashinfer_fp8_blockscale_gemm_fake`. Required fake/meta implementation for torch.compile graph tracing. Internally it relies on calls such as `empty`.
**CN:** 该函数 `_flashinfer_fp8_blockscale_gemm_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 flashinfer fp 8 blockscale gemm fake 相关工作。 它内部会调用 `empty` 等例程。

### Top-level block (lines 218-222)
```python
direct_register_custom_op(
    "flashinfer_fp8_blockscale_gemm",
    _flashinfer_fp8_blockscale_gemm_impl,
    fake_impl=_flashinfer_fp8_blockscale_gemm_fake,
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Function `_dynamic_flashinfer_deepgemm_blockscale_gemm_impl` (lines 225-309)
```python
def _dynamic_flashinfer_deepgemm_blockscale_gemm_impl(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    group_size: int,
    use_deep_gemm_e8m0: bool,
) -> torch.Tensor:
    """
    Conditional FlashInfer FP8 blockscale GEMM with batch-size-dependent selection.

    This function switches between two optimized kernels based on the input batch size:
    - For small batches (M < 32): Uses FlashInfer's DeepGEMM swapAB optimization.
    - For larger batches (M >= 32): Uses the official DeepGEMM kernel.

    The conditional logic must use torch.cond() instead of a simple if-else statement
    to maintain compatibility with torch.compile graph compilation.

    This batch-size-dependent selection is essential for maintaining model accuracy.
    Benchmarks on GSM8K show a significant accuracy gap (88% vs 95%) for DeepSeek-V3.1
    when using FlashInfer's DeepGEMM on M>=32. The M < 32 strategy fixes the accuracy
    drop.

    Args:
        input: Input tensor of shape (batch_size, input_dim) in FP8 format
        weight: Weight tensor of shape (output_dim, input_dim) in FP8 format
        weight_scale: Scale factors for weight quantization (per-group)
        group_size: Quantization group size for the weight tensor
        use_deep_gemm_e8m0: Whether to use the E8M0 format in DeepGEMM quantization

    Returns:
        Output tensor of shape (batch_size, output_dim) in bfloat16 format
    """

    def run_flashinfer_deepgemm_swapAB(
        input: torch.Tensor,
        weight: torch.Tensor,
        weight_scale: torch.Tensor,
    ) -> torch.Tensor:
        return flashinfer_fp8_blockscale_gemm(
            input=input,
            weight=weight,
            weight_scale=weight_scale,
            out_dtype=torch.bfloat16,
        )

    def run_deepgemm(
        input: torch.Tensor,
        weight: torch.Tensor,
        weight_scale: torch.Tensor,
    ) -> torch.Tensor:
        q_input, input_scale = per_token_group_quant_fp8(
            input,
            group_size=group_size,
            column_major_scales=True,
            use_ue8m0=use_deep_gemm_e8m0,
        )
        output = torch.empty(
            (q_input.shape[0], weight.shape[0]),
            dtype=torch.bfloat16,
            device=q_input.device,
        )
        fp8_gemm_nt(
            (q_input, input_scale),
            (weight, weight_scale),
            output,
            is_deep_gemm_e8m0_used=use_deep_gemm_e8m0,
        )
        return output

    if envs.VLLM_BATCH_INVARIANT:
        return run_deepgemm(input, weight, weight_scale)

    condition = input.shape[0] < 32

    # PyTorch's torch.compile cannot handle input-dependent control flow in standard
    # Python conditionals. torch.cond() explicitly registers both code paths in the
    # computation graph, allowing torch.compile to capture both branches.
    # without torch.cond, the M < 32 condition won't be able to be captured by torch
    # compile
    return torch.cond(
        condition,
        run_flashinfer_deepgemm_swapAB,
        run_deepgemm,
        (input, weight, weight_scale),
    )
```
**EN:** This helper implements `_dynamic_flashinfer_deepgemm_blockscale_gemm_impl`. Conditional FlashInfer FP8 blockscale GEMM with batch-size-dependent selection. Internally it relies on calls such as `cond`, `flashinfer_fp8_blockscale_gemm`, `per_token_group_quant_fp8`, `empty`.
**CN:** 该函数 `_dynamic_flashinfer_deepgemm_blockscale_gemm_impl` 封装了此模块中的一段关键运行时逻辑，重点处理 dynamic flashinfer deepgemm blockscale gemm impl 相关工作。 它内部会调用 `cond`, `flashinfer_fp8_blockscale_gemm`, `per_token_group_quant_fp8`, `empty` 等例程。

### Function `_dynamic_flashinfer_deepgemm_blockscale_gemm_fake` (lines 312-324)
```python
def _dynamic_flashinfer_deepgemm_blockscale_gemm_fake(
    input: torch.Tensor,
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
    group_size: int,
    use_deep_gemm_e8m0: bool,
) -> torch.Tensor:
    """
    Required fake/meta implementation for torch.compile graph tracing.
    """
    return torch.empty(
        input.shape[0], weight.shape[0], dtype=torch.bfloat16, device=input.device
    )
```
**EN:** This helper implements `_dynamic_flashinfer_deepgemm_blockscale_gemm_fake`. Required fake/meta implementation for torch.compile graph tracing. Internally it relies on calls such as `empty`.
**CN:** 该函数 `_dynamic_flashinfer_deepgemm_blockscale_gemm_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 dynamic flashinfer deepgemm blockscale gemm fake 相关工作。 它内部会调用 `empty` 等例程。

### Top-level block (lines 327-331)
```python
direct_register_custom_op(
    "dynamic_flashinfer_deepgemm_blockscale_gemm",
    _dynamic_flashinfer_deepgemm_blockscale_gemm_impl,
    fake_impl=_dynamic_flashinfer_deepgemm_blockscale_gemm_fake,
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

## Key Concepts / 关键概念
- **FlashInfer backend / FlashInfer 后端**
  - **EN:** The code integrates FlashInfer primitives for fast low-precision inference paths.
  - **CN:** 代码集成 FlashInfer 原语，以支持高效低精度推理路径。
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
- **vLLM internal / vLLM 内部依赖**: `vllm.envs`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.flashinfer`, `vllm.utils.torch_utils`, `.BlockScaledMMLinearKernel`, `.deep_gemm`, `.ScaledMMLinearKernel`
- **External / 外部依赖**: `typing`, `torch`
