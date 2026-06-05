# gate_linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/gate_linear.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-8 — imports and setup
```python
import torch
from torch.nn.parameter import Parameter

from vllm.model_executor.custom_op import PluggableLayer
from vllm.model_executor.layers.linear import ReplicatedLinear
from vllm.platforms import current_platform
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 11-117 — class `GateLinear`
```python
@PluggableLayer.register("gate_linear")
class GateLinear(ReplicatedLinear):
    """MoE gate linear layer with three-tier GEMM dispatch:

    1. DSV3 specialized kernel (SM90+, batch<=16, supported dims)
    2. cuBLAS bf16×bf16→fp32 (SM90+ + bf16 + fp32 out_dtype)
    3. F.linear via ReplicatedLinear (ultimate fallback)

    The ``out_dtype`` attribute is mutable and can be set after init
    (e.g. when the required dtype depends on the expert quantization
    method which is only known later).
    """

    # Dimensions supported by the DSV3 specialized kernel
# ... omitted for brevity ...
            output = output.to(self.out_dtype)
        return output, output_bias
```
**EN:** This class defines `GateLinear`. It inherits from `ReplicatedLinear`. MoE gate linear layer with three-tier GEMM dispatch: 1. Important methods include `__init__`, `set_out_dtype`, `forward`. Key calls include `PluggableLayer.register`, `super.__init__`, `super.forward`, `current_platform.is_device_capability`, `current_platform.is_device_capability_family`, `current_platform.is_cuda`. It writes or updates `DSV3_SUPPORTED_NUM_EXPERTS`, `DSV3_SUPPORTED_HIDDEN_SIZES`, `is_hopper_or_blackwell`, `can_use_specialized_kernels`, `out_dtype`, `allow_specialized_router_gemm`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `GateLinear`。 它继承自 `ReplicatedLinear`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `set_out_dtype`, `forward`。 关键调用包括 `PluggableLayer.register`, `super.__init__`, `super.forward`, `current_platform.is_device_capability`, `current_platform.is_device_capability_family`, `current_platform.is_cuda`。 它会写入或更新 `DSV3_SUPPORTED_NUM_EXPERTS`, `DSV3_SUPPORTED_HIDDEN_SIZES`, `is_hopper_or_blackwell`, `can_use_specialized_kernels`, `out_dtype`, `allow_specialized_router_gemm`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 28-73 — method `GateLinear.__init__`
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        bias: bool = False,
        out_dtype: torch.dtype | None = None,
        params_dtype: torch.dtype | None = None,
        force_fp32_compute: bool = False,
        prefix: str = "",
    ):
        is_hopper_or_blackwell = current_platform.is_device_capability(
            (9, 0)
        ) or current_platform.is_device_capability_family(100)
        can_use_specialized_kernels = (
            current_platform.is_cuda() and is_hopper_or_blackwell and not bias
        )

        # If fp32 compute is required and no specialized kernel is available,
        # store weights in fp32 so Tier 3 computes in fp32 natively.
        if force_fp32_compute and not can_use_specialized_kernels:
            params_dtype = torch.float32

        super().__init__(
            input_size,
            output_size,
            bias=bias,
            params_dtype=params_dtype,
            quant_config=None,
            prefix=prefix,
        )
        self.out_dtype = out_dtype

        # DSV3 specialized kernel eligibility (SM90+, exact dims)
        self.allow_specialized_router_gemm = can_use_specialized_kernels
        self.allow_dsv3_router_gemm = (
            self.allow_specialized_router_gemm
            and output_size in self.DSV3_SUPPORTED_NUM_EXPERTS
            and input_size in self.DSV3_SUPPORTED_HIDDEN_SIZES
        )

        # cuBLAS bf16→fp32 eligibility
        self.allow_cublas_router_gemm = (
            self.allow_specialized_router_gemm
            and self.weight.dtype == torch.bfloat16
            and self.out_dtype == torch.float32
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `input_size`, `output_size`, `bias`, `out_dtype`, `params_dtype`, `force_fp32_compute`. Key calls include `super.__init__`, `current_platform.is_device_capability`, `current_platform.is_device_capability_family`, `current_platform.is_cuda`, `super`. It writes or updates `is_hopper_or_blackwell`, `can_use_specialized_kernels`, `out_dtype`, `allow_specialized_router_gemm`, `allow_dsv3_router_gemm`, `allow_cublas_router_gemm`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `input_size`, `output_size`, `bias`, `out_dtype`, `params_dtype`, `force_fp32_compute`。 关键调用包括 `super.__init__`, `current_platform.is_device_capability`, `current_platform.is_device_capability_family`, `current_platform.is_cuda`, `super`。 它会写入或更新 `is_hopper_or_blackwell`, `can_use_specialized_kernels`, `out_dtype`, `allow_specialized_router_gemm`, `allow_dsv3_router_gemm`, `allow_cublas_router_gemm`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 75-90 — method `GateLinear.set_out_dtype`
```python
    def set_out_dtype(self, out_dtype: torch.dtype) -> None:
        """Set output dtype for the router logits after init.

        Useful when the required dtype depends on the expert quantization
        method which is only known after the gate is constructed.
        """
        if self.out_dtype is not None:
            raise ValueError("out_dtype has already been set")
        self.out_dtype = out_dtype

        if (
            not self.allow_cublas_router_gemm
            and self.allow_specialized_router_gemm
            and out_dtype == torch.float32
        ):
            self.allow_cublas_router_gemm = self.weight.dtype == torch.bfloat16
```
**EN:** This method defines `set_out_dtype`. Set output dtype for the router logits after init. The main inputs are `out_dtype`. Key calls include `ValueError`. It writes or updates `out_dtype`, `allow_cublas_router_gemm`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `set_out_dtype`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `out_dtype`。 关键调用包括 `ValueError`。 它会写入或更新 `out_dtype`, `allow_cublas_router_gemm`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 92-117 — method `GateLinear.forward`
```python
    def forward(
        self, x: torch.Tensor
    ) -> torch.Tensor | tuple[torch.Tensor, Parameter | None]:
        import vllm._custom_ops as ops

        # Tier 1: DSV3 specialized kernel
        if self.allow_dsv3_router_gemm and x.shape[0] <= 16:
            output = ops.dsv3_router_gemm(
                hidden_states=x,
                router_weight=self.weight,
                output_dtype=self.out_dtype,
            )
            return output, None

        # Tier 2: cuBLAS bf16→fp32
        if self.allow_cublas_router_gemm and x.dtype == torch.bfloat16:
            output = torch.mm(x, self.weight.T, out_dtype=torch.float32)
            return output, None

        # Tier 3: F.linear (ReplicatedLinear)
        if self.out_dtype is not None and x.dtype != self.weight.dtype:
            x = x.to(self.weight.dtype)
        output, output_bias = super().forward(x)
        if self.out_dtype is not None and output.dtype != self.out_dtype:
            output = output.to(self.out_dtype)
        return output, output_bias
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `x`. Key calls include `super.forward`, `ops.dsv3_router_gemm`, `torch.mm`, `x.to`, `output.to`, `super`. It writes or updates `output`, `output_bias`, `x`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `x`。 关键调用包括 `super.forward`, `ops.dsv3_router_gemm`, `torch.mm`, `x.to`, `output.to`, `super`。 它会写入或更新 `output`, `output_bias`, `x`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `GateLinear` / [CN] 核心符号：`GateLinear`

## Dependencies / 依赖关系
- **External**: `torch`, `torch.nn.parameter` / **外部依赖**: `torch`, `torch.nn.parameter`
- **Internal**: `vllm.model_executor.custom_op`, `vllm.model_executor.layers.linear`, `vllm.platforms` / **内部依赖**: `vllm.model_executor.custom_op`, `vllm.model_executor.layers.linear`, `vllm.platforms`
- **Runtime traits**: platform-aware dispatch / **运行时特征**: platform-aware dispatch
