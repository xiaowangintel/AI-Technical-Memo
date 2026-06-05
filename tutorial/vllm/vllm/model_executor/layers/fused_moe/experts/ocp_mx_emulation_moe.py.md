# ocp_mx_emulation_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/ocp_mx_emulation_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: OCP MX quantization emulation for MoE. / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 14-31 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.experts.triton_moe import TritonExperts
from vllm.model_executor.layers.fused_moe.utils import moe_kernel_quantize_input
from vllm.model_executor.layers.quantization.utils.mxfp4_utils import dequant_mxfp4
from vllm.model_executor.layers.quantization.utils.mxfp6_utils import dequant_mxfp6
from vllm.model_executor.layers.quantization.utils.ocp_mx_utils import (
    OCP_MX_Scheme,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 34-186 — class `OCP_MXQuantizationEmulationTritonExperts`
```python
class OCP_MXQuantizationEmulationTritonExperts(TritonExperts):
    """
    Extension of TritonExperts to support emulated OCP MX MoE experts.

    It may be used for OCP MX (MXFP4/MXFP6) models when the device does not
    have native support for these dtypes.
    """

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(moe_config, quant_config)
# ... omitted for brevity ...
            apply_router_weight_on_input=apply_router_weight_on_input,
        )
```
**EN:** This class defines `OCP_MXQuantizationEmulationTritonExperts`. It inherits from `TritonExperts`. Extension of TritonExperts to support emulated OCP MX MoE experts. Important methods include `__init__`, `quant_dtype`, `expects_unquantized_inputs`, `_supports_quant_scheme`, `_dequantize_weights`, `apply`. Key calls include `super.__init__`, `logger.warning_once`, `self.ocp_mx_scheme.startswith`, `self._dequantize_weights`, `moe_kernel_quantize_input`, `super.apply`. It writes or updates `ocp_mx_scheme`, `w1_scale_val`, `w2_scale_val`, `scale`, `quantization_emulation`, `w1_dequant`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `OCP_MXQuantizationEmulationTritonExperts`。 它继承自 `TritonExperts`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `quant_dtype`, `expects_unquantized_inputs`, `_supports_quant_scheme`, `_dequantize_weights`, `apply`。 关键调用包括 `super.__init__`, `logger.warning_once`, `self.ocp_mx_scheme.startswith`, `self._dequantize_weights`, `moe_kernel_quantize_input`, `super.apply`。 它会写入或更新 `ocp_mx_scheme`, `w1_scale_val`, `w2_scale_val`, `scale`, `quantization_emulation`, `w1_dequant`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 42-88 — method `OCP_MXQuantizationEmulationTritonExperts.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
    ):
        super().__init__(moe_config, quant_config)
        logger.warning_once(
            "Using OCP_MXQuantizationEmulationTritonExperts MOE backend. This"
            " will dequantize weights on the fly and may be slower than native"
            " quantized MOE. Consider using a device with native OCP MX"
            " quantization support for better performance."
        )

        self.ocp_mx_scheme = quant_config.ocp_mx_scheme
        assert self.ocp_mx_scheme is not None, (
            "ocp_mx_scheme must be set in quant_config for"
            " OCP_MXQuantizationEmulationTritonExperts"
        )

        # `TritonExperts.apply` expects pre-dequantized weights,
        # which we handle in `apply` below.
        self.w1_scale_val = self.quant_config.w1_scale
        self.w2_scale_val = self.quant_config.w2_scale

        self.quant_config._w1.scale = None
        self.quant_config._w2.scale = None

        self.quantization_emulation = True

        if self.ocp_mx_scheme in {
            OCP_MX_Scheme.w_mxfp4_a_mxfp4,
        }:
            # Weight has to be dequantized for mxfp4 emulation.
            self._quant_dtype = "mxfp4"
        elif self.ocp_mx_scheme in [
            OCP_MX_Scheme.w_mxfp4_a_mxfp6_e3m2,
            OCP_MX_Scheme.w_mxfp4_a_mxfp6_e2m3,
            OCP_MX_Scheme.w_mxfp6_e3m2_a_mxfp6_e3m2,
            OCP_MX_Scheme.w_mxfp6_e2m3_a_mxfp6_e2m3,
        ]:
            self._quant_dtype = "mxfp6"
        elif self.ocp_mx_scheme in [
            OCP_MX_Scheme.w_mxfp4_a_fp8,
            OCP_MX_Scheme.w_mxfp6_e3m2_a_fp8,
        ]:
            # TODO: double check this one
            self._quant_dtype = "mxfp8"
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`. Key calls include `super.__init__`, `logger.warning_once`, `super`. It writes or updates `ocp_mx_scheme`, `w1_scale_val`, `w2_scale_val`, `scale`, `quantization_emulation`, `_quant_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`。 关键调用包括 `super.__init__`, `logger.warning_once`, `super`。 它会写入或更新 `ocp_mx_scheme`, `w1_scale_val`, `w2_scale_val`, `scale`, `quantization_emulation`, `_quant_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 90-92 — method `OCP_MXQuantizationEmulationTritonExperts.quant_dtype`
```python
    @property
    def quant_dtype(self) -> torch.dtype | str | None:
        return self._quant_dtype
```
**EN:** This method defines `quant_dtype`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `quant_dtype`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 94-96 — method `OCP_MXQuantizationEmulationTritonExperts.expects_unquantized_inputs`
```python
    @property
    def expects_unquantized_inputs(self) -> bool:
        return True
```
**EN:** This method defines `expects_unquantized_inputs`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `expects_unquantized_inputs`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 98-105 — method `OCP_MXQuantizationEmulationTritonExperts._supports_quant_scheme`
```python
    @staticmethod
    def _supports_quant_scheme(
        weight_key,
        activation_key,
    ) -> bool:
        # This class is used for emulation only - the oracle selects it
        # directly rather than via quant scheme matching.
        return True
```
**EN:** This method defines `_supports_quant_scheme`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `weight_key`, `activation_key`.
**CN:** 该方法定义 `_supports_quant_scheme`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `weight_key`, `activation_key`。

### Lines 107-121 — method `OCP_MXQuantizationEmulationTritonExperts._dequantize_weights`
```python
    def _dequantize_weights(
        self,
        w: torch.Tensor,
        w_scale: torch.Tensor,
        dtype: torch.dtype,
    ) -> torch.Tensor:
        """Dequantize weights based on the OCP MX scheme."""
        if self.ocp_mx_scheme.startswith("w_mxfp4"):  # type: ignore[union-attr]
            return dequant_mxfp4(w, w_scale, dtype)
        elif self.ocp_mx_scheme.startswith("w_mxfp6_e3m2"):  # type: ignore[union-attr]
            return dequant_mxfp6(w, w_scale, quant_dtype="fp6_e3m2", float_dtype=dtype)
        elif self.ocp_mx_scheme.startswith("w_mxfp6_e2m3"):  # type: ignore[union-attr]
            return dequant_mxfp6(w, w_scale, quant_dtype="fp6_e2m3", float_dtype=dtype)
        else:
            raise NotImplementedError(f"Unsupported ocp_mx_scheme={self.ocp_mx_scheme}")
```
**EN:** This method defines `_dequantize_weights`. Dequantize weights based on the OCP MX scheme. The main inputs are `w`, `w_scale`, `dtype`. Key calls include `self.ocp_mx_scheme.startswith`, `dequant_mxfp4`, `dequant_mxfp6`, `NotImplementedError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `_dequantize_weights`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `w`, `w_scale`, `dtype`。 关键调用包括 `self.ocp_mx_scheme.startswith`, `dequant_mxfp4`, `dequant_mxfp6`, `NotImplementedError`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 123-186 — method `OCP_MXQuantizationEmulationTritonExperts.apply`
```python
    def apply(
        self,
        output: torch.Tensor,
        hidden_states: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        activation: MoEActivation,
        global_num_experts: int,
        expert_map: torch.Tensor | None,
        a1q_scale: torch.Tensor | None,
        a2_scale: torch.Tensor | None,
        workspace13: torch.Tensor,
        workspace2: torch.Tensor,
        expert_tokens_meta: mk.ExpertTokensMetadata | None,
        apply_router_weight_on_input: bool,
    ):
        """
        Apply emulated quantized MoE computation.

        This dequantizes the weights on the fly and calls TritonExperts.apply
        with activation quantization support.
        """
        assert w1.dtype == torch.uint8
        assert w2.dtype == torch.uint8

        # Dequantize w1 and w2 from packed OCP MX format to bf16/fp16
        w1_dequant = self._dequantize_weights(
            w1, self.w1_scale_val, hidden_states.dtype
        )
        w2_dequant = self._dequantize_weights(
            w2, self.w2_scale_val, hidden_states.dtype
        )

        # Apply activation QDQ if needed by the OCP MX scheme
        hidden_states, _ = moe_kernel_quantize_input(
            A=hidden_states,
            A_scale=None,
            quant_dtype=self.quant_config.quant_dtype,
            per_act_token_quant=False,
            ocp_mx_scheme=self.ocp_mx_scheme,
            quantization_emulation=True,
        )
# ... omitted for brevity ...
            apply_router_weight_on_input=apply_router_weight_on_input,
        )
```
**EN:** This method defines `apply`. Apply emulated quantized MoE computation. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `self._dequantize_weights`, `moe_kernel_quantize_input`, `super.apply`, `super`. It writes or updates `w1_dequant`, `w2_dequant`, `hidden_states`, `_`.
**CN:** 该方法定义 `apply`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `self._dequantize_weights`, `moe_kernel_quantize_input`, `super.apply`, `super`。 它会写入或更新 `w1_dequant`, `w2_dequant`, `hidden_states`, `_`。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `OCP_MXQuantizationEmulationTritonExperts` / [CN] 核心符号：`OCP_MXQuantizationEmulationTritonExperts`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.triton_moe`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils`, `vllm.model_executor.layers.quantization.utils.mxfp6_utils`, `vllm.model_executor.layers.quantization.utils.ocp_mx_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.triton_moe`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.mxfp4_utils`, `vllm.model_executor.layers.quantization.utils.mxfp6_utils`, `vllm.model_executor.layers.quantization.utils.ocp_mx_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
