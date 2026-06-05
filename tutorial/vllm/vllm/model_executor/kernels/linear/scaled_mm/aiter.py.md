# aiter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/aiter.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the AITER scaled matrix-multiplication kernel path. / 实现 AITER 的缩放矩阵乘内核路径。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-26)
```python
import torch

from vllm import _custom_ops as ops
from vllm._aiter_ops import (
    rocm_aiter_ops,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
)
from vllm.model_executor.utils import replace_parameter
from vllm.platforms import current_platform

from .BlockScaledMMLinearKernel import (
    Fp8BlockScaledMMLinearKernel,
)
from .cutlass import CutlassInt8ScaledMMLinearKernel
from .ScaledMMLinearKernel import (
    FP8ScaledMMLinearKernel,
    FP8ScaledMMLinearLayerConfig,
    Int8ScaledMMLinearLayerConfig,
)
```
**EN:** This import block loads `torch`, `vllm`, `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `aiter.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm`, `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`, ...，为 `aiter.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 28-28)
```python
logger = init_logger(__name__)
```
**EN:** This assignment block initializes `logger`, capturing shared state or configuration consumed by later helper functions and kernel classes.
**CN:** 该赋值代码块初始化了 `logger`，保存后续辅助函数和内核类会消费的共享状态或配置。

### Class `AiterInt8ScaledMMLinearKernel` (lines 31-125)
```python
class AiterInt8ScaledMMLinearKernel(CutlassInt8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_rocm():
            return False, "Requires ROCm."

        if compute_capability is not None and compute_capability < 90:
            return False, "requires compute capability 90 and above."

        try:
            import aiter  # noqa: F401 # deliberately attempt to import aiter
        except Exception:
            return False, "requires `aiter` to be installed."

        if not rocm_aiter_ops.is_linear_enabled():
            return (
                False,
                "requires setting `VLLM_ROCM_USE_AITER=1` "
                "and `VLLM_ROCM_USE_AITER_LINEAR=1`. "
                "`VLLM_ROCM_USE_AITER_LINEAR` default is True.",
            )
        return True, None

    @classmethod
    def can_implement(cls, c: Int8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        if not c.input_symmetric:
            return False, "supports symmetric quantization only."
        return True, None

    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """
        `AiterInt8ScaledMMLinearKernel` implements a fused version of
            `output = torch.mm((scale_a * a), (scale_b * b)).to(out_dtype)`
        where scale_a * a and scale_b * b are implemented using numpy-style
        broadcasting.
        Currently only support per-tensor-per-tensor GEMM
        and per-token-per-channel GEMM through AITER
        w8a8 scaled gemm. `AiterInt8ScaledMMLinearKernel` also does not support
        ATIER block scaled GEMM and mix-precision GEMM.
        """
        w_q, w_s, i_s, i_zp, azp_adj = self._get_layer_params(layer)

        # ops.scaled_int8_quant supports both dynamic and static quant:
        # * dynamic, i_s is None and x_s computed from x.
        # * static, i_s is scalar and x_s is i_s.
        symmetric = azp_adj is None
        assert symmetric, (
            "AiterInt8ScaledMMLinearKernel only supports symmetric quantization."
        )
        x_q, x_s, x_zp = ops.scaled_int8_quant(x, i_s, i_zp, symmetric=symmetric)

        assert x_zp is None, (
            "AiterInt8ScaledMMLinearKernel only supports symmetric quantization."
        )
        out_dtype = x.dtype

        assert w_q.shape[0] % 16 == 0 and w_q.shape[1] % 16 == 0
        assert out_dtype is torch.bfloat16 or out_dtype is torch.float16
        assert bias is None or bias.shape[0] == w_q.shape[1] and bias.dtype == out_dtype

        m = x_q.shape[0]  # a
        n = w_q.shape[1]  # b

        per_tensor_scale_a = x_s.numel() == 1
        per_tensor_scale_b = w_s.numel() == 1
        per_token_scale_a = x_s.numel() == m
        per_channel_scale_b = w_s.numel() == n

        # @TODO:
        # Maybe broadcast the per-tensor-scale into per-channel-scale
        # if one of the scale is a per-channel-scale.
        # For now, it only supports:
        # - per-tensor-per-tensor a8w8 scaled GEMM, and
        # - per-token-per-channel a8w8 scaled GEMM
        assert (per_tensor_scale_a and per_tensor_scale_b) or (
            per_token_scale_a and per_channel_scale_b
        ), (
            "Currently only support per-tensor-per-tensor GEMM "
            " and per-token-per-channel GEMM through AITER"
            " w8a8 scaled gemm. `AiterInt8ScaledMMLinearKernel` "
            "does not support AITER block scaled GEMM."
        )

        # gemm_a8w8_CK(a, b, scale_a, scale_b, bias) expects
        # a to be [M, K]
        # b to be [N, K]
        # CutlassInt8ScaledMMLinearKernel prepare weight `w_q` in [K, N] format
        return rocm_aiter_ops.w8a8_gemm(x_q, w_q.t(), x_s, w_s, bias, out_dtype)
```
**EN:** This kernel class defines `AiterInt8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `CutlassInt8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `apply_weights`.
**CN:** 该内核类定义了 `AiterInt8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `CutlassInt8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `apply_weights`。

### Method `AiterInt8ScaledMMLinearKernel.is_supported` (lines 33-54)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_rocm():
            return False, "Requires ROCm."

        if compute_capability is not None and compute_capability < 90:
            return False, "requires compute capability 90 and above."

        try:
            import aiter  # noqa: F401 # deliberately attempt to import aiter
        except Exception:
            return False, "requires `aiter` to be installed."

        if not rocm_aiter_ops.is_linear_enabled():
            return (
                False,
                "requires setting `VLLM_ROCM_USE_AITER=1` "
                "and `VLLM_ROCM_USE_AITER_LINEAR=1`. "
                "`VLLM_ROCM_USE_AITER_LINEAR` default is True.",
            )
        return True, None
```
**EN:** This method implements `AiterInt8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_rocm`, `is_linear_enabled`.
**CN:** 该方法 `AiterInt8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_rocm`, `is_linear_enabled` 等例程。

### Method `AiterInt8ScaledMMLinearKernel.can_implement` (lines 57-60)
```python
    def can_implement(cls, c: Int8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        if not c.input_symmetric:
            return False, "supports symmetric quantization only."
        return True, None
```
**EN:** This method implements `AiterInt8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module.
**CN:** 该方法 `AiterInt8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。

### Method `AiterInt8ScaledMMLinearKernel.apply_weights` (lines 62-125)
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """
        `AiterInt8ScaledMMLinearKernel` implements a fused version of
            `output = torch.mm((scale_a * a), (scale_b * b)).to(out_dtype)`
        where scale_a * a and scale_b * b are implemented using numpy-style
        broadcasting.
        Currently only support per-tensor-per-tensor GEMM
        and per-token-per-channel GEMM through AITER
        w8a8 scaled gemm. `AiterInt8ScaledMMLinearKernel` also does not support
        ATIER block scaled GEMM and mix-precision GEMM.
        """
        w_q, w_s, i_s, i_zp, azp_adj = self._get_layer_params(layer)

        # ops.scaled_int8_quant supports both dynamic and static quant:
        # * dynamic, i_s is None and x_s computed from x.
        # * static, i_s is scalar and x_s is i_s.
        symmetric = azp_adj is None
        assert symmetric, (
            "AiterInt8ScaledMMLinearKernel only supports symmetric quantization."
        )
        x_q, x_s, x_zp = ops.scaled_int8_quant(x, i_s, i_zp, symmetric=symmetric)

        assert x_zp is None, (
            "AiterInt8ScaledMMLinearKernel only supports symmetric quantization."
        )
        out_dtype = x.dtype

        assert w_q.shape[0] % 16 == 0 and w_q.shape[1] % 16 == 0
        assert out_dtype is torch.bfloat16 or out_dtype is torch.float16
        assert bias is None or bias.shape[0] == w_q.shape[1] and bias.dtype == out_dtype

        m = x_q.shape[0]  # a
        n = w_q.shape[1]  # b

        per_tensor_scale_a = x_s.numel() == 1
        per_tensor_scale_b = w_s.numel() == 1
        per_token_scale_a = x_s.numel() == m
        per_channel_scale_b = w_s.numel() == n

        # @TODO:
        # Maybe broadcast the per-tensor-scale into per-channel-scale
        # if one of the scale is a per-channel-scale.
        # For now, it only supports:
        # - per-tensor-per-tensor a8w8 scaled GEMM, and
        # - per-token-per-channel a8w8 scaled GEMM
        assert (per_tensor_scale_a and per_tensor_scale_b) or (
            per_token_scale_a and per_channel_scale_b
        ), (
            "Currently only support per-tensor-per-tensor GEMM "
            " and per-token-per-channel GEMM through AITER"
            " w8a8 scaled gemm. `AiterInt8ScaledMMLinearKernel` "
            "does not support AITER block scaled GEMM."
        )

        # gemm_a8w8_CK(a, b, scale_a, scale_b, bias) expects
        # a to be [M, K]
        # b to be [N, K]
        # CutlassInt8ScaledMMLinearKernel prepare weight `w_q` in [K, N] format
        return rocm_aiter_ops.w8a8_gemm(x_q, w_q.t(), x_s, w_s, bias, out_dtype)
```
**EN:** This method implements `AiterInt8ScaledMMLinearKernel.apply_weights`. `AiterInt8ScaledMMLinearKernel` implements a fused version of `output = torch.mm((scale_a * a), (scale_b * b)).to(out_dtype)` where scale_a * a and scale_b * b are implemented using numpy-style broadcasting. Internally it relies on calls such as `_get_layer_params`, `scaled_int8_quant`, `w8a8_gemm`, `numel`.
**CN:** 该方法 `AiterInt8ScaledMMLinearKernel.apply_weights` 封装了此模块中的一段关键运行时逻辑，重点处理 apply weights 相关工作。 它内部会调用 `_get_layer_params`, `scaled_int8_quant`, `w8a8_gemm`, `numel` 等例程。

### Class `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel` (lines 128-212)
```python
class AiterPreshuffledPerTokenFp8ScaledMMLinearKernel(FP8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_rocm():
            return False, "requires ROCm."
        if not rocm_aiter_ops.is_linear_fp8_enabled():
            return (
                False,
                "requires setting `VLLM_ROCM_USE_AITER=1` "
                "and `VLLM_ROCM_USE_AITER_LINEAR=1`. "
                "`VLLM_ROCM_USE_AITER_LINEAR` default is True.",
            )
        try:
            import aiter  # noqa: F401
        except Exception:
            return False, "requires aiter library to be installed."
        return True, None

    @classmethod
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        is_ptpc = (
            c.activation_quant_key.scale.group_shape.is_per_token()
            and c.weight_quant_key.scale.group_shape.is_per_channel()
        )
        if c.weight_shape is None:
            return False, "weight_shape is required for Aiter kernels"
        N, K = c.weight_shape
        fp8_dtype = current_platform.fp8_dtype()

        if c.out_dtype is not torch.bfloat16:
            return False, "requires bfloat16 output dtype."

        if not is_ptpc:
            return (
                False,
                "requires per token activation scales and per channel weight scales.",
            )

        if not (N % 16 == 0 and K % 16 == 0):
            return (
                False,
                f"requires N and K dimensions divisible by 16, received "
                f"N={N} and K={K}.",
            )

        # Aiter's shuffled per-token Gemm performs better than torch only when its
        # tuned.
        if not rocm_aiter_ops.is_shuffled_per_token_w8a8_gemm_tuned(N, K, fp8_dtype):
            return (
                False,
                f"requires a tuned configuration for N: {N} and K: {K} "
                f"and fp8 dtype {fp8_dtype}.",
            )

        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_name, *_ = self.layer_param_names
        w, *_ = self._get_layer_params(layer)

        replace_parameter(
            layer,
            w_name,
            torch.nn.Parameter(
                rocm_aiter_ops.shuffle_weight(w.t().contiguous()).data,
                requires_grad=False,
            ),
        )

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
        return rocm_aiter_ops.preshuffled_per_token_w8a8_gemm(
            A, B, As, Bs, bias, out_dtype
        )
```
**EN:** This kernel class defines `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `FP8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_scaled_mm`.
**CN:** 该内核类定义了 `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `FP8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_scaled_mm`。

### Method `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.is_supported` (lines 130-146)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        if not current_platform.is_rocm():
            return False, "requires ROCm."
        if not rocm_aiter_ops.is_linear_fp8_enabled():
            return (
                False,
                "requires setting `VLLM_ROCM_USE_AITER=1` "
                "and `VLLM_ROCM_USE_AITER_LINEAR=1`. "
                "`VLLM_ROCM_USE_AITER_LINEAR` default is True.",
            )
        try:
            import aiter  # noqa: F401
        except Exception:
            return False, "requires aiter library to be installed."
        return True, None
```
**EN:** This method implements `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_rocm`, `is_linear_fp8_enabled`.
**CN:** 该方法 `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_rocm`, `is_linear_fp8_enabled` 等例程。

### Method `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.can_implement` (lines 149-184)
```python
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        is_ptpc = (
            c.activation_quant_key.scale.group_shape.is_per_token()
            and c.weight_quant_key.scale.group_shape.is_per_channel()
        )
        if c.weight_shape is None:
            return False, "weight_shape is required for Aiter kernels"
        N, K = c.weight_shape
        fp8_dtype = current_platform.fp8_dtype()

        if c.out_dtype is not torch.bfloat16:
            return False, "requires bfloat16 output dtype."

        if not is_ptpc:
            return (
                False,
                "requires per token activation scales and per channel weight scales.",
            )

        if not (N % 16 == 0 and K % 16 == 0):
            return (
                False,
                f"requires N and K dimensions divisible by 16, received "
                f"N={N} and K={K}.",
            )

        # Aiter's shuffled per-token Gemm performs better than torch only when its
        # tuned.
        if not rocm_aiter_ops.is_shuffled_per_token_w8a8_gemm_tuned(N, K, fp8_dtype):
            return (
                False,
                f"requires a tuned configuration for N: {N} and K: {K} "
                f"and fp8 dtype {fp8_dtype}.",
            )

        return True, None
```
**EN:** This method implements `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `fp8_dtype`, `is_per_token`, `is_per_channel`, `is_shuffled_per_token_w8a8_gemm_tuned`.
**CN:** 该方法 `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `fp8_dtype`, `is_per_token`, `is_per_channel`, `is_shuffled_per_token_w8a8_gemm_tuned` 等例程。

### Method `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.process_weights_after_loading` (lines 186-197)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_name, *_ = self.layer_param_names
        w, *_ = self._get_layer_params(layer)

        replace_parameter(
            layer,
            w_name,
            torch.nn.Parameter(
                rocm_aiter_ops.shuffle_weight(w.t().contiguous()).data,
                requires_grad=False,
            ),
        )
```
**EN:** This method implements `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_layer_params`, `replace_parameter`, `Parameter`, `shuffle_weight`.
**CN:** 该方法 `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `_get_layer_params`, `replace_parameter`, `Parameter`, `shuffle_weight` 等例程。

### Method `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.apply_scaled_mm` (lines 199-212)
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
        return rocm_aiter_ops.preshuffled_per_token_w8a8_gemm(
            A, B, As, Bs, bias, out_dtype
        )
```
**EN:** This method implements `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.apply_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `preshuffled_per_token_w8a8_gemm`.
**CN:** 该方法 `AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.apply_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply scaled mm 相关工作。 它内部会调用 `preshuffled_per_token_w8a8_gemm` 等例程。

### Class `AiterPerTokenFp8ScaledMMLinearKernel` (lines 215-272)
```python
class AiterPerTokenFp8ScaledMMLinearKernel(FP8ScaledMMLinearKernel):
    @classmethod
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        return AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.is_supported(
            compute_capability
        )

    @classmethod
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        is_ptpc = (
            c.activation_quant_key.scale.group_shape.is_per_token()
            and c.weight_quant_key.scale.group_shape.is_per_channel()
        )
        if c.weight_shape is None:
            return False, "weight_shape is required for Aiter kernels"
        N, K = c.weight_shape
        fp8_dtype = current_platform.fp8_dtype()

        if not is_ptpc:
            return (
                False,
                "requires per token activation scales and per channel weight scales.",
            )

        # Aiter's per-token Gemm performs better than torch oonly when its
        # tuned.
        if not rocm_aiter_ops.is_per_token_w8a8_gemm_tuned(N, K, fp8_dtype):
            return (
                False,
                f"requires a tuned configuration for N: {N} and K: {K} "
                f"and fp8 dtype {fp8_dtype}.",
            )
        return True, None

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_name, *_ = self.layer_param_names
        w, *_ = self._get_layer_params(layer)

        replace_parameter(
            layer,
            w_name,
            torch.nn.Parameter(w.t(), requires_grad=False),
        )

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
        return rocm_aiter_ops.w8a8_gemm(A, B, As, Bs, bias, out_dtype)
```
**EN:** This kernel class defines `AiterPerTokenFp8ScaledMMLinearKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `FP8ScaledMMLinearKernel`. Key methods include `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_scaled_mm`.
**CN:** 该内核类定义了 `AiterPerTokenFp8ScaledMMLinearKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `FP8ScaledMMLinearKernel`。 关键方法包括 `is_supported`, `can_implement`, `process_weights_after_loading`, `apply_scaled_mm`。

### Method `AiterPerTokenFp8ScaledMMLinearKernel.is_supported` (lines 217-222)
```python
    def is_supported(
        cls, compute_capability: int | None = None
    ) -> tuple[bool, str | None]:
        return AiterPreshuffledPerTokenFp8ScaledMMLinearKernel.is_supported(
            compute_capability
        )
```
**EN:** This method implements `AiterPerTokenFp8ScaledMMLinearKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_supported`.
**CN:** 该方法 `AiterPerTokenFp8ScaledMMLinearKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_supported` 等例程。

### Method `AiterPerTokenFp8ScaledMMLinearKernel.can_implement` (lines 225-249)
```python
    def can_implement(cls, c: FP8ScaledMMLinearLayerConfig) -> tuple[bool, str | None]:
        is_ptpc = (
            c.activation_quant_key.scale.group_shape.is_per_token()
            and c.weight_quant_key.scale.group_shape.is_per_channel()
        )
        if c.weight_shape is None:
            return False, "weight_shape is required for Aiter kernels"
        N, K = c.weight_shape
        fp8_dtype = current_platform.fp8_dtype()

        if not is_ptpc:
            return (
                False,
                "requires per token activation scales and per channel weight scales.",
            )

        # Aiter's per-token Gemm performs better than torch oonly when its
        # tuned.
        if not rocm_aiter_ops.is_per_token_w8a8_gemm_tuned(N, K, fp8_dtype):
            return (
                False,
                f"requires a tuned configuration for N: {N} and K: {K} "
                f"and fp8 dtype {fp8_dtype}.",
            )
        return True, None
```
**EN:** This method implements `AiterPerTokenFp8ScaledMMLinearKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `fp8_dtype`, `is_per_token`, `is_per_channel`, `is_per_token_w8a8_gemm_tuned`.
**CN:** 该方法 `AiterPerTokenFp8ScaledMMLinearKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `fp8_dtype`, `is_per_token`, `is_per_channel`, `is_per_token_w8a8_gemm_tuned` 等例程。

### Method `AiterPerTokenFp8ScaledMMLinearKernel.process_weights_after_loading` (lines 251-259)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w_name, *_ = self.layer_param_names
        w, *_ = self._get_layer_params(layer)

        replace_parameter(
            layer,
            w_name,
            torch.nn.Parameter(w.t(), requires_grad=False),
        )
```
**EN:** This method implements `AiterPerTokenFp8ScaledMMLinearKernel.process_weights_after_loading`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `_get_layer_params`, `replace_parameter`, `Parameter`, `t`.
**CN:** 该方法 `AiterPerTokenFp8ScaledMMLinearKernel.process_weights_after_loading` 封装了此模块中的一段关键运行时逻辑，重点处理 process weights after loading 相关工作。 它内部会调用 `_get_layer_params`, `replace_parameter`, `Parameter`, `t` 等例程。

### Method `AiterPerTokenFp8ScaledMMLinearKernel.apply_scaled_mm` (lines 261-272)
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
        return rocm_aiter_ops.w8a8_gemm(A, B, As, Bs, bias, out_dtype)
```
**EN:** This method implements `AiterPerTokenFp8ScaledMMLinearKernel.apply_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `w8a8_gemm`.
**CN:** 该方法 `AiterPerTokenFp8ScaledMMLinearKernel.apply_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply scaled mm 相关工作。 它内部会调用 `w8a8_gemm` 等例程。

### Class `AiterFp8BlockScaledMMKernel` (lines 275-338)
```python
class AiterFp8BlockScaledMMKernel(Fp8BlockScaledMMLinearKernel):
    def __init__(self, config: FP8ScaledMMLinearLayerConfig):
        super().__init__(config)
        n, k = config.weight_shape

        self.use_triton = (
            not current_platform.is_fp8_fnuz()
            and rocm_aiter_ops.is_triton_gemm_w8a8_tuned(n, k)
        )

    @classmethod
    def is_supported(cls, compute_capability=None):
        return (
            rocm_aiter_ops.is_linear_enabled(),
            "Only supported on ROCm platform \
                with aiter package installed.",
        )

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
        return True, None

    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        if As.dtype != Bs.dtype:
            from vllm.model_executor.layers.quantization.utils.fp8_utils import (
                _upcast_e8m0_to_fp32,
            )

            if As.dtype == torch.float8_e8m0fnu:
                As = _upcast_e8m0_to_fp32(As).contiguous()
            else:
                As = As.to(torch.float32)

            if Bs.dtype == torch.float8_e8m0fnu:
                Bs = _upcast_e8m0_to_fp32(Bs).contiguous()
            else:
                Bs = Bs.to(torch.float32)

        out_dtype = self.config.out_dtype
        if self.use_triton:
            gemm_a8w8_blockscale_op = rocm_aiter_ops.triton_gemm_a8w8_blockscale
        else:
            gemm_a8w8_blockscale_op = rocm_aiter_ops.gemm_a8w8_blockscale

        return gemm_a8w8_blockscale_op(
            A, B, As, Bs, list(self.weight_group_shape), output_dtype=out_dtype
        )
```
**EN:** This kernel class defines `AiterFp8BlockScaledMMKernel`. It organizes the state and behavior needed by this kernel path. It inherits from `Fp8BlockScaledMMLinearKernel`. Key methods include `__init__`, `is_supported`, `can_implement`, `apply_block_scaled_mm`.
**CN:** 该内核类定义了 `AiterFp8BlockScaledMMKernel`。 它组织了该内核路径所需的状态与行为。 它继承自 `Fp8BlockScaledMMLinearKernel`。 关键方法包括 `__init__`, `is_supported`, `can_implement`, `apply_block_scaled_mm`。

### Method `AiterFp8BlockScaledMMKernel.__init__` (lines 276-283)
```python
    def __init__(self, config: FP8ScaledMMLinearLayerConfig):
        super().__init__(config)
        n, k = config.weight_shape

        self.use_triton = (
            not current_platform.is_fp8_fnuz()
            and rocm_aiter_ops.is_triton_gemm_w8a8_tuned(n, k)
        )
```
**EN:** This method implements `AiterFp8BlockScaledMMKernel.__init__`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `__init__`, `is_triton_gemm_w8a8_tuned`, `super`, `is_fp8_fnuz`.
**CN:** 该方法 `AiterFp8BlockScaledMMKernel.__init__` 封装了此模块中的一段关键运行时逻辑，重点处理 init 相关工作。 它内部会调用 `__init__`, `is_triton_gemm_w8a8_tuned`, `super`, `is_fp8_fnuz` 等例程。

### Method `AiterFp8BlockScaledMMKernel.is_supported` (lines 286-291)
```python
    def is_supported(cls, compute_capability=None):
        return (
            rocm_aiter_ops.is_linear_enabled(),
            "Only supported on ROCm platform \
                with aiter package installed.",
        )
```
**EN:** This method implements `AiterFp8BlockScaledMMKernel.is_supported`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `is_linear_enabled`.
**CN:** 该方法 `AiterFp8BlockScaledMMKernel.is_supported` 封装了此模块中的一段关键运行时逻辑，重点处理 is supported 相关工作。 它内部会调用 `is_linear_enabled` 等例程。

### Method `AiterFp8BlockScaledMMKernel.can_implement` (lines 294-306)
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
        return True, None
```
**EN:** This method implements `AiterFp8BlockScaledMMKernel.can_implement`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `can_implement`, `GroupShape`, `super`.
**CN:** 该方法 `AiterFp8BlockScaledMMKernel.can_implement` 封装了此模块中的一段关键运行时逻辑，重点处理 can implement 相关工作。 它内部会调用 `can_implement`, `GroupShape`, `super` 等例程。

### Method `AiterFp8BlockScaledMMKernel.apply_block_scaled_mm` (lines 308-338)
```python
    def apply_block_scaled_mm(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        As: torch.Tensor,
        Bs: torch.Tensor,
    ) -> torch.Tensor:
        if As.dtype != Bs.dtype:
            from vllm.model_executor.layers.quantization.utils.fp8_utils import (
                _upcast_e8m0_to_fp32,
            )

            if As.dtype == torch.float8_e8m0fnu:
                As = _upcast_e8m0_to_fp32(As).contiguous()
            else:
                As = As.to(torch.float32)

            if Bs.dtype == torch.float8_e8m0fnu:
                Bs = _upcast_e8m0_to_fp32(Bs).contiguous()
            else:
                Bs = Bs.to(torch.float32)

        out_dtype = self.config.out_dtype
        if self.use_triton:
            gemm_a8w8_blockscale_op = rocm_aiter_ops.triton_gemm_a8w8_blockscale
        else:
            gemm_a8w8_blockscale_op = rocm_aiter_ops.gemm_a8w8_blockscale

        return gemm_a8w8_blockscale_op(
            A, B, As, Bs, list(self.weight_group_shape), output_dtype=out_dtype
        )
```
**EN:** This method implements `AiterFp8BlockScaledMMKernel.apply_block_scaled_mm`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `gemm_a8w8_blockscale_op`, `list`, `contiguous`, `to`.
**CN:** 该方法 `AiterFp8BlockScaledMMKernel.apply_block_scaled_mm` 封装了此模块中的一段关键运行时逻辑，重点处理 apply block scaled mm 相关工作。 它内部会调用 `gemm_a8w8_blockscale_op`, `list`, `contiguous`, `to` 等例程。

## Key Concepts / 关键概念
- **Triton kernels / Triton 内核**
  - **EN:** The implementation relies on Triton-generated GPU kernels for low-level tensor work.
  - **CN:** 该实现依赖 Triton 生成的 GPU 内核完成底层张量计算。
- **CUTLASS backend / CUTLASS 后端**
  - **EN:** The module selects or wraps CUTLASS kernels for NVIDIA-oriented matrix multiplication.
  - **CN:** 该模块选择或封装 CUTLASS 内核，以支持面向 NVIDIA 的矩阵乘计算。
- **AITER backend / AITER 后端**
  - **EN:** The module routes execution through AITER custom operators or kernel helpers.
  - **CN:** 该模块通过 AITER 自定义算子或内核辅助逻辑完成执行。
- **ROCm support / ROCm 支持**
  - **EN:** The implementation targets ROCm-capable devices and their kernel constraints.
  - **CN:** 该实现面向 ROCm 设备及其内核约束。
- **FP8 quantization / FP8 量化**
  - **EN:** The module handles FP8 data, scales, or kernels for low-precision inference.
  - **CN:** 该模块处理 FP8 数据、缩放因子或相关低精度推理内核。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm._aiter_ops`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.utils`, `vllm.platforms`, `.BlockScaledMMLinearKernel`, `.cutlass`, `.ScaledMMLinearKernel`
- **External / 外部依赖**: `torch`
