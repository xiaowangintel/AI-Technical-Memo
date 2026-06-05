# test_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_fusion, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_fusion 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-55)
```python
import pytest
import torch

import vllm.config
import vllm.ir.ops
import vllm.plugins
from tests.compile.backend import TestBackend
from tests.utils import TestFP8Layer
from vllm._aiter_ops import IS_AITER_FOUND, rocm_aiter_ops
from vllm.compilation.passes.fusion.matcher_utils import QUANT_OPS
from vllm.compilation.passes.fusion.rms_quant_fusion import (
    FUSED_OPS,
    FusedRMSQuantKey,
    RMSNormQuantFusionPass,
)
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    ModelConfig,
    PassConfig,
    VllmConfig,
)
from vllm.model_executor.kernels.linear import (
    AiterFp8BlockScaledMMKernel,
    ChannelWiseTorchFP8ScaledMMLinearKernel,
    CutlassFp8BlockScaledMMKernel,
    CutlassFP8ScaledMMLinearKernel,
    DeepGemmFp8BlockScaledMMKernel,
    FlashInferFp8DeepGEMMDynamicBlockScaledKernel,
    FlashInferFP8ScaledMMLinearKernel,
    PerTensorTorchFP8ScaledMMLinearKernel,
    ROCmFP8ScaledMMLinearKernel,
    RowWiseTorchFP8ScaledMMLinearKernel,
    TritonFp8BlockScaledMMKernel,
    _KernelT,
)
from vllm.model_executor.layers.layernorm import RMSNorm, RMSNormGated
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    create_fp8_quant_key,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    cutlass_block_fp8_supported,
)
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import (
    is_deep_gemm_e8m0_used,
    is_deep_gemm_supported,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.compile.backend, tests.utils; and vLLM components like vllm.config, vllm.ir.ops, vllm.plugins, vllm._aiter_ops.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.compile.backend、tests.utils；vLLM 内部组件，例如 vllm.config、vllm.ir.ops、vllm.plugins、vllm._aiter_ops。

### Constants and module state (lines 57-110)
```python
FP8_DTYPE = current_platform.fp8_dtype()

RMS_ADD_OP = torch.ops._C.fused_add_rms_norm.default

# Kernel and group_shape combinations: (kernel, group_shape)
# CUDA kernels
CUDA_KERNEL_GROUPSHAPE_COMBINATIONS = [
    # FlashInferFP8ScaledMMLinearKernel supports both per-tensor only
    (FlashInferFP8ScaledMMLinearKernel, GroupShape.PER_TENSOR),
    # CutlassFP8ScaledMMLinearKernel supports both per-tensor and per-token
    (CutlassFP8ScaledMMLinearKernel, GroupShape.PER_TOKEN),
    (CutlassFP8ScaledMMLinearKernel, GroupShape.PER_TENSOR),
    # PerTensorTorchFP8ScaledMMLinearKernel only supports per-tensor
    (PerTensorTorchFP8ScaledMMLinearKernel, GroupShape.PER_TENSOR),
    # ChannelWiseTorchFP8ScaledMMLinearKernel only supports per-token
    (ChannelWiseTorchFP8ScaledMMLinearKernel, GroupShape.PER_TOKEN),
    # Blockwise group shapes
    (FlashInferFp8DeepGEMMDynamicBlockScaledKernel, GroupShape(1, 128)),
    (CutlassFp8BlockScaledMMKernel, GroupShape(1, 128)),
    (DeepGemmFp8BlockScaledMMKernel, GroupShape(1, 128)),
    (TritonFp8BlockScaledMMKernel, GroupShape(1, 128)),
    (TritonFp8BlockScaledMMKernel, GroupShape(1, 64)),
]

# ROCm kernels
ROCM_KERNEL_GROUPSHAPE_COMBINATIONS = [
    # ROCmFP8ScaledMMLinearKernel supports per-tensor only
    (ROCmFP8ScaledMMLinearKernel, GroupShape.PER_TENSOR),
    # RowWiseTorchFP8ScaledMMLinearKernel only supports per-token
    (RowWiseTorchFP8ScaledMMLinearKernel, GroupShape.PER_TOKEN),
    # ChannelWiseTorchFP8ScaledMMLinearKernel only supports per-token
    (ChannelWiseTorchFP8ScaledMMLinearKernel, GroupShape.PER_TOKEN),
    # Blockwise group shapes (no kernel abstraction)
    (TritonFp8BlockScaledMMKernel, GroupShape(1, 128)),
    (TritonFp8BlockScaledMMKernel, GroupShape(1, 64)),
]

KERNEL_GROUPSHAPE_COMBINATIONS = (
    CUDA_KERNEL_GROUPSHAPE_COMBINATIONS
    if current_platform.is_cuda()
    else ROCM_KERNEL_GROUPSHAPE_COMBINATIONS
)

# For Aiter tests we toggle use_aiter_quant_op
AITER_KERNEL_GROUPSHAPE_COMBINATIONS = [
    # Per-token with RowWiseTorchFP8ScaledMMLinearKernel
    (RowWiseTorchFP8ScaledMMLinearKernel, GroupShape.PER_TOKEN, True),
    (RowWiseTorchFP8ScaledMMLinearKernel, GroupShape.PER_TOKEN, False),
    # Per-token with ChannelWiseTorchFP8ScaledMMLinearKernel
    (ChannelWiseTorchFP8ScaledMMLinearKernel, GroupShape.PER_TOKEN, True),
    (ChannelWiseTorchFP8ScaledMMLinearKernel, GroupShape.PER_TOKEN, False),
    # Blockwise
    (AiterFp8BlockScaledMMKernel, GroupShape(1, 128), True),
]
```
**EN:** This block centralizes shared constants and parameter grids, including FP8_DTYPE, RMS_ADD_OP, CUDA_KERNEL_GROUPSHAPE_COMBINATIONS, ROCM_KERNEL_GROUPSHAPE_COMBINATIONS, KERNEL_GROUPSHAPE_COMBINATIONS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FP8_DTYPE、RMS_ADD_OP、CUDA_KERNEL_GROUPSHAPE_COMBINATIONS、ROCM_KERNEL_GROUPSHAPE_COMBINATIONS、KERNEL_GROUPSHAPE_COMBINATIONS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `TestModel` (lines 113-113)
```python
class TestModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestModel.__init__` (lines 114-173)
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float,
        force_kernel: type[_KernelT] | None,
        group_shape: GroupShape,
        dtype: torch.dtype,
        use_aiter_fusion: bool = False,
        use_aiter_quant: bool = False,
        *args,
        **kwargs,
    ):
        super().__init__(*args, **kwargs)
        self.fp8_linear_layers: list[torch.nn.Module]
        self.group_shape = group_shape
        self.use_aiter_quant_op = use_aiter_quant
        self.use_aiter_fusion = use_aiter_fusion
        self.norm = [RMSNorm(hidden_size, eps) for _ in range(4)]
        self.enable_rms_norm_custom_op = self.norm[0].enabled()

        # Determine if blockwise based on group_shape
        is_blockwise = group_shape.is_per_group()

        if is_blockwise:
            block_size = group_shape.col
            self.activation_quant_key = create_fp8_quant_key(
                static=False, group_shape=group_shape
            )
            self.weight_quant_key = create_fp8_quant_key(
                static=True, group_shape=GroupShape(block_size, block_size)
            )

        else:
            is_static = group_shape == GroupShape.PER_TENSOR
            self.activation_quant_key = create_fp8_quant_key(
                is_static, group_shape=group_shape
            )
            self.weight_quant_key = create_fp8_quant_key(
                static=True, group_shape=group_shape
            )

        self.fp8_linear_layers = [
            TestFP8Layer(
                weight_shape=(hidden_size, hidden_size),
                activation_quant_key=self.activation_quant_key,
                weight_quant_key=self.weight_quant_key,
                force_kernel=force_kernel,
                transpose_weights=use_aiter_fusion,
                input_dtype=dtype,
            )
            for _ in range(3)
        ]

        # Enable aiter quantization if requested
        for layer in self.fp8_linear_layers:
            layer.kernel.quant_fp8.use_aiter = use_aiter_quant

        self.enable_quant_fp8_custom_op = self.fp8_linear_layers[
            0
        ].is_quant_fp8_enabled()
```
**EN:** This method implements the initialization for `TestModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestModel.forward` (lines 175-191)
```python
    def forward(self, x):
        # avoid having graph input be an arg to a pattern directly
        x = resid = torch.relu(x)
        y = self.norm[0](x)

        x2 = self.fp8_linear_layers[0](y)
        # make sure resid is used for replacement to work
        y2, resid = self.norm[1](x2, resid)

        x3 = self.fp8_linear_layers[1](y2)

        y3, resid = self.norm[2](x3, resid)  # use resid here

        x4 = self.fp8_linear_layers[2](y3)

        y4, resid = self.norm[3](x4, resid)  # use resid here
        return y4
```
**EN:** This method on `TestModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestModel.ops_in_model_before` (lines 193-209)
```python
    def ops_in_model_before(self):
        if self.group_shape.is_per_group():
            # Blockwise path
            if self.use_aiter_fusion and self.use_aiter_quant_op:
                return [rocm_aiter_ops.get_group_quant_op()]
            if self.use_aiter_fusion:
                return [torch.ops.vllm.triton_per_token_group_quant_fp8.default]
        else:
            if self.use_aiter_quant_op:
                return [rocm_aiter_ops.get_per_token_quant_op()]

        # Common path
        return (
            [QUANT_OPS[self.activation_quant_key]]
            if self.enable_quant_fp8_custom_op
            else [torch.ops.aten.reciprocal]
        )
```
**EN:** This method on `TestModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestModel.ops_in_model_after` (lines 211-240)
```python
    def ops_in_model_after(self):
        if self.use_aiter_fusion:
            if self.group_shape.is_per_group():
                # Blockwise aiter fusion
                from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
                    AiterFusedAddRMSFp8GroupQuantPattern,
                    AiterRMSFp8GroupQuantPattern,
                )

                return [
                    AiterFusedAddRMSFp8GroupQuantPattern.FUSED_OP,
                    AiterRMSFp8GroupQuantPattern.FUSED_OP,
                ]
            else:
                # Per-token aiter fusion
                from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
                    AiterFusedAddRMSNormDynamicQuantPattern,
                    AiterRMSNormDynamicQuantPattern,
                )

                return [
                    AiterFusedAddRMSNormDynamicQuantPattern.FUSED_OP,
                    AiterRMSNormDynamicQuantPattern.FUSED_OP,
                ]

        # Regular fusion
        return [
            FUSED_OPS[FusedRMSQuantKey(self.activation_quant_key, True)],
            FUSED_OPS[FusedRMSQuantKey(self.activation_quant_key, False)],
        ]
```
**EN:** This method on `TestModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestModel.ops_in_model_before_partial` (lines 242-246)
```python
    def ops_in_model_before_partial(self):
        return [
            torch.ops.vllm_ir.rms_norm,
            torch.ops.vllm_ir.fused_add_rms_norm.default,
        ]
```
**EN:** This method on `TestModel` implements ops in model before partial. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestModel` 中的这个方法实现了 ops in model before partial。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_run_fusion_test` (lines 249-287)
```python
def _run_fusion_test(
    model,
    fusion_pass,
    vllm_config,
    dtype,
    hidden_size,
    num_tokens,
):
    """Helper function for common fusion test logic.

    Must be called within vllm_config context.
    """
    noop_pass = NoOpEliminationPass(vllm_config)
    cleanup_pass = PostCleanupPass(vllm_config)

    backend = TestBackend(noop_pass, fusion_pass, cleanup_pass)
    backend2 = TestBackend(noop_pass, cleanup_pass)

    x = torch.rand(num_tokens, hidden_size)
    torch._dynamo.mark_dynamic(x, 0)

    model_fused = torch.compile(model, backend=backend)
    result_fused = model_fused(x)

    model_unfused = torch.compile(model, backend=backend2)
    result_unfused = model_unfused(x)

    if dtype == torch.float16:
        ATOL, RTOL = (2e-3, 2e-3)
    else:
        ATOL, RTOL = (1e-2, 1e-2)

    torch.testing.assert_close(result_fused, result_unfused, atol=ATOL, rtol=RTOL)

    assert fusion_pass.matched_count == 3
    backend.check_before_ops(model.ops_in_model_before())
    backend.check_after_ops(model.ops_in_model_after())

    return backend, backend2
```
**EN:** This helper function implements the shared logic for run fusion test. numeric results are compared against a reference with explicit tolerances.
**CN:** 该辅助函数实现了 run fusion test 所需的共享逻辑。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_fusion_rmsnorm_quant` (lines 290-382)
```python
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
@pytest.mark.parametrize("hidden_size", [256])
@pytest.mark.parametrize("num_tokens", [257])
@pytest.mark.parametrize("eps", [1e-5, 1e-6])
@pytest.mark.parametrize("kernel_groupshape", KERNEL_GROUPSHAPE_COMBINATIONS)
@pytest.mark.parametrize("enable_rms_norm_custom_op", [True, False])
@pytest.mark.parametrize("enable_quant_fp8_custom_op", [True, False])
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(), reason="Only test on CUDA and ROCm"
)
def test_fusion_rmsnorm_quant(
    dtype,
    hidden_size,
    num_tokens,
    eps,
    kernel_groupshape,
    enable_rms_norm_custom_op,
    enable_quant_fp8_custom_op,
):
    force_kernel, group_shape = kernel_groupshape

    if not enable_quant_fp8_custom_op and group_shape.is_per_group():
        pytest.skip("Unsupported unwrapped quant fp8 op for blockwise quantization")

    if group_shape == GroupShape(1, 64) and (
        cutlass_block_fp8_supported() or is_deep_gemm_supported()
    ):
        pytest.skip("Unsupported group shape 64 for CUTLASS/DeepGemm")

    # TODO(quant-rms-fusion): DeepGEMM UE8M0 activation quant on B200 lowers
    # to a packed int32-scale op (per_token_group_quant_fp8_packed_for_deepgemm),
    # but the rms+quant fusion pattern only matches the fp32-scale variant, so
    # the fused output gets a mismatched scale layout and produces NaN. Only
    # reproduces on bf16 (DeepGEMM UE8M0 on B200 is bf16-only).
    # To re-enable: make rms_norm_per_block_quant emit packed UE8M0 scales
    # and extend the fusion pattern to rewrite the packed activation quant.
    deepgemm_kernels = (
        DeepGemmFp8BlockScaledMMKernel,
        FlashInferFp8DeepGEMMDynamicBlockScaledKernel,
    )
# ... excerpt ...

    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=dtype),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=custom_ops,
            pass_config=PassConfig(
                fuse_norm_quant=True, fuse_act_quant=True, eliminate_noops=True
            ),
        ),
    )

    with (
        vllm.config.set_current_vllm_config(vllm_config),
        vllm_config.kernel_config.ir_op_priority.set_priority(),
    ):
        # Setup device before model creation
        torch.set_default_device("cuda")
        torch.set_default_dtype(dtype)
        torch.manual_seed(1)

        fusion_pass = RMSNormQuantFusionPass(vllm_config)

        model = TestModel(
            hidden_size=hidden_size,
            eps=eps,
            force_kernel=force_kernel,
            group_shape=group_shape,
            dtype=dtype,
            use_aiter_fusion=False,
            use_aiter_quant=False,
        )

        backend, _ = _run_fusion_test(
            model, fusion_pass, vllm_config, dtype, hidden_size, num_tokens
        )
        backend.check_before_ops(
            model.ops_in_model_before_partial(), fully_replaced=False
        )
```
**EN:** This pytest case verifies fusion rmsnorm quant. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dtype, hidden_size, num_tokens, eps. unsupported hardware, backend, or configuration combinations are skipped early. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 fusion rmsnorm quant 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 dtype、hidden_size、num_tokens、eps 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_aiter_fusion_rmsnorm_quant` (lines 385-441)
```python
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("hidden_size", [256])
@pytest.mark.parametrize("num_tokens", [257])
@pytest.mark.parametrize("eps", [1e-5, 1e-6])
@pytest.mark.parametrize(
    "kernel_groupshape_quant", AITER_KERNEL_GROUPSHAPE_COMBINATIONS
)
@pytest.mark.skipif(
    (not current_platform.is_rocm() or not IS_AITER_FOUND),
    reason="Only test on ROCm with aiter package installed",
)
def test_aiter_fusion_rmsnorm_quant(
    dtype: torch.dtype,
    hidden_size: int,
    num_tokens: int,
    eps: float,
    kernel_groupshape_quant: tuple,
    monkeypatch: pytest.MonkeyPatch,
):
    force_kernel, group_shape, use_aiter_quant_op = kernel_groupshape_quant
    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=dtype),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=["+rms_norm", "+quant_fp8"],
            pass_config=PassConfig(fuse_norm_quant=True, eliminate_noops=True),
        ),
    )

    with vllm.config.set_current_vllm_config(vllm_config), monkeypatch.context() as m:
        from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
            RocmAiterRMSNormQuantFusionPass,
        )

        m.setenv("VLLM_ROCM_USE_AITER", "1")

        rocm_aiter_ops.refresh_env_variables()

        torch.set_default_device("cuda")
        torch.set_default_dtype(dtype)
        torch.manual_seed(1)

        fusion_pass = RocmAiterRMSNormQuantFusionPass(vllm_config)

        model = TestModel(
            hidden_size=hidden_size,
            eps=eps,
            force_kernel=force_kernel,
            group_shape=group_shape,
            dtype=dtype,
            use_aiter_fusion=True,  # Always use aiter fusion ops in aiter test
            use_aiter_quant=use_aiter_quant_op,  # Toggle aiter quantization
        )

        _run_fusion_test(
            model, fusion_pass, vllm_config, dtype, hidden_size, num_tokens
        )
```
**EN:** This pytest case verifies aiter fusion rmsnorm quant. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dtype, hidden_size, num_tokens, eps. unsupported hardware, backend, or configuration combinations are skipped early. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 aiter fusion rmsnorm quant 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 dtype、hidden_size、num_tokens、eps 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会控制随机性以保证场景可复现。

### Class `TestGatedModel` (lines 444-451)
```python
class TestGatedModel(torch.nn.Module):
    """Model that uses RMSNormGated + reshape + group FP8 quant + linear.

    Mimics GatedDeltaNetAttention's output projection path where:
    - RMSNormGated operates on per-head tensors (N*H, D)
    - Output is reshaped to (N, H*D) before group quantization + linear
    """
```
**EN:** This helper class groups the state and behavior needed for TestGatedModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestGatedModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestGatedModel.__init__` (lines 452-489)
```python
    def __init__(
        self,
        num_heads: int,
        head_dim: int,
        eps: float,
        force_kernel: type[_KernelT],
        group_shape: GroupShape,
        dtype: torch.dtype,
        use_aiter_quant: bool = True,
    ):
        super().__init__()
        self.num_heads = num_heads
        self.head_dim = head_dim
        hidden_dim = num_heads * head_dim

        self.norm = RMSNormGated(
            head_dim,
            eps=eps,
            group_size=None,
            norm_before_gate=True,
        )

        self.activation_quant_key = create_fp8_quant_key(
            static=False, group_shape=group_shape
        )
        self.weight_quant_key = create_fp8_quant_key(
            static=True, group_shape=GroupShape(group_shape.col, group_shape.col)
        )

        self.fp8_linear = TestFP8Layer(
            weight_shape=(hidden_dim, hidden_dim),
            activation_quant_key=self.activation_quant_key,
            weight_quant_key=self.weight_quant_key,
            force_kernel=force_kernel,
            transpose_weights=True,
            input_dtype=dtype,
        )
        self.fp8_linear.kernel.quant_fp8.use_aiter = use_aiter_quant
```
**EN:** This method implements the initialization for `TestGatedModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestGatedModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestGatedModel.forward` (lines 491-502)
```python
    def forward(self, x, z):
        num_heads = self.num_heads
        head_dim = self.head_dim
        hidden_dim = num_heads * head_dim
        x = torch.relu(x)
        z = torch.relu(z)
        x_heads = x.reshape(-1, num_heads, head_dim).reshape(-1, head_dim)
        z_heads = z.reshape(-1, num_heads, head_dim).reshape(-1, head_dim)
        normed = self.norm(x_heads, z_heads)
        merged = normed.reshape(-1, hidden_dim)
        out = self.fp8_linear(merged)
        return out
```
**EN:** This method on `TestGatedModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestGatedModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestGatedModel.ops_in_model_after` (lines 504-509)
```python
    def ops_in_model_after(self):
        from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
            AiterRMSNormGatedFp8GroupQuantPattern,
        )

        return [AiterRMSNormGatedFp8GroupQuantPattern.FUSED_OP]
```
**EN:** This method on `TestGatedModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestGatedModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `_MockGDNLayer` (lines 512-518)
```python
class _MockGDNLayer:
    """Minimal mock to populate static_forward_context for pass discovery.

    Uses __class__ assignment to pass isinstance checks against
    GatedDeltaNetAttention without requiring a full config-based init.
    """
```
**EN:** This helper class groups the state and behavior needed for MockGDNLayer. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MockGDNLayer 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `_MockGDNLayer.__init__` (lines 519-528)
```python
    def __init__(self, num_v_heads: int, head_v_dim: int, tp_size: int = 1):
        self.num_v_heads = num_v_heads
        self.head_v_dim = head_v_dim
        self.tp_size = tp_size

        from vllm.model_executor.layers.mamba.gdn_linear_attn import (
            GatedDeltaNetAttention,
        )

        self.__class__ = GatedDeltaNetAttention
```
**EN:** This method implements the initialization for `_MockGDNLayer`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `_MockGDNLayer` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_aiter_fusion_rmsnorm_gated_quant` (lines 531-609)
```python
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("num_heads", [2])
@pytest.mark.parametrize("head_dim", [128])
@pytest.mark.parametrize("num_tokens", [8])
@pytest.mark.parametrize("eps", [1e-5, 1e-6])
@pytest.mark.skipif(
    (not current_platform.is_rocm() or not IS_AITER_FOUND),
    reason="Only test on ROCm with aiter package installed",
)
def test_aiter_fusion_rmsnorm_gated_quant(
    dtype: torch.dtype,
    num_heads: int,
    head_dim: int,
    num_tokens: int,
    eps: float,
    monkeypatch: pytest.MonkeyPatch,
):
    group_shape = GroupShape(1, 128)
    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=dtype),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=["-rms_norm", "-silu_and_mul", "-quant_fp8"],
            pass_config=PassConfig(fuse_norm_quant=True, eliminate_noops=True),
        ),
    )

    with vllm.config.set_current_vllm_config(vllm_config), monkeypatch.context() as m:
        from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
            RocmAiterRMSNormQuantFusionPass,
        )

        m.setenv("VLLM_ROCM_USE_AITER", "1")
        rocm_aiter_ops.refresh_env_variables()

        # Register a mock GDN layer so the pass discovers num_heads/head_dim
        mock_gdn = _MockGDNLayer(num_v_heads=num_heads, head_v_dim=head_dim, tp_size=1)
        vllm_config.compilation_config.static_forward_context["mock_gdn_layer"] = (
            mock_gdn
        )

        torch.set_default_device("cuda")
        torch.set_default_dtype(dtype)
        torch.manual_seed(1)

        fusion_pass = RocmAiterRMSNormQuantFusionPass(vllm_config)

        model = TestGatedModel(
            num_heads=num_heads,
            head_dim=head_dim,
            eps=eps,
            force_kernel=AiterFp8BlockScaledMMKernel,
            group_shape=group_shape,
            dtype=dtype,
            use_aiter_quant=True,
        )

        noop_pass = NoOpEliminationPass(vllm_config)
        cleanup_pass = PostCleanupPass(vllm_config)

        backend = TestBackend(noop_pass, fusion_pass, cleanup_pass)
        backend2 = TestBackend(noop_pass, cleanup_pass)

        hidden_dim = num_heads * head_dim
        x = torch.rand(num_tokens, hidden_dim)
        z = torch.rand(num_tokens, hidden_dim)
        torch._dynamo.mark_dynamic(x, 0)
        torch._dynamo.mark_dynamic(z, 0)

        model_fused = torch.compile(model, backend=backend)
        result_fused = model_fused(x, z)

        model_unfused = torch.compile(model, backend=backend2)
        result_unfused = model_unfused(x, z)

        torch.testing.assert_close(result_fused, result_unfused, atol=1e-2, rtol=1e-2)

        assert fusion_pass.matched_count == 1
        backend.check_after_ops(model.ops_in_model_after())
```
**EN:** This pytest case verifies aiter fusion rmsnorm gated quant. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dtype, num_heads, head_dim, num_tokens. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 aiter fusion rmsnorm gated quant 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 dtype、num_heads、head_dim、num_tokens 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_aiter_fusion_rmsnorm_gated_quant_no_gdn_layers` (lines 612-680)
```python
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("num_heads", [2])
@pytest.mark.parametrize("head_dim", [128])
@pytest.mark.parametrize("num_tokens", [8])
@pytest.mark.parametrize("eps", [1e-6])
@pytest.mark.skipif(
    (not current_platform.is_rocm() or not IS_AITER_FOUND),
    reason="Only test on ROCm with aiter package installed",
)
def test_aiter_fusion_rmsnorm_gated_quant_no_gdn_layers(
    dtype: torch.dtype,
    num_heads: int,
    head_dim: int,
    num_tokens: int,
    eps: float,
    monkeypatch: pytest.MonkeyPatch,
):
    """Verify that without GDN layers in static_forward_context,
    the gated pattern is not registered and no matches occur."""
    group_shape = GroupShape(1, 128)
    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=dtype),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=["-rms_norm", "-silu_and_mul", "-quant_fp8"],
            pass_config=PassConfig(fuse_norm_quant=True, eliminate_noops=True),
        ),
    )

    with vllm.config.set_current_vllm_config(vllm_config), monkeypatch.context() as m:
        from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
            RocmAiterRMSNormQuantFusionPass,
        )

        m.setenv("VLLM_ROCM_USE_AITER", "1")
        rocm_aiter_ops.refresh_env_variables()

        torch.set_default_device("cuda")
        torch.set_default_dtype(dtype)
        torch.manual_seed(1)

        # No mock GDN layer registered -- pass should not register gated pattern
        fusion_pass = RocmAiterRMSNormQuantFusionPass(vllm_config)

        model = TestGatedModel(
            num_heads=num_heads,
            head_dim=head_dim,
            eps=eps,
            force_kernel=AiterFp8BlockScaledMMKernel,
            group_shape=group_shape,
            dtype=dtype,
            use_aiter_quant=True,
        )

        noop_pass = NoOpEliminationPass(vllm_config)
        cleanup_pass = PostCleanupPass(vllm_config)

        backend = TestBackend(noop_pass, fusion_pass, cleanup_pass)

        hidden_dim = num_heads * head_dim
        x = torch.rand(num_tokens, hidden_dim)
        z = torch.rand(num_tokens, hidden_dim)
        torch._dynamo.mark_dynamic(x, 0)
        torch._dynamo.mark_dynamic(z, 0)

        model_fused = torch.compile(model, backend=backend)
        model_fused(x, z)

        assert fusion_pass.matched_count == 0
```
**EN:** This pytest case verifies aiter fusion rmsnorm gated quant no gdn layers. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dtype, num_heads, head_dim, num_tokens. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 aiter fusion rmsnorm gated quant no gdn layers 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 dtype、num_heads、head_dim、num_tokens 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.config`
- `vllm.ir.ops`
- `vllm.plugins`
- `tests.compile.backend -> TestBackend`
- `tests.utils -> TestFP8Layer`
- `vllm._aiter_ops -> IS_AITER_FOUND, rocm_aiter_ops`
- `vllm.compilation.passes.fusion.matcher_utils -> QUANT_OPS`
- `vllm.compilation.passes.fusion.rms_quant_fusion -> FUSED_OPS, FusedRMSQuantKey, RMSNormQuantFusionPass`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.config -> CompilationConfig, CompilationMode, ModelConfig, PassConfig, VllmConfig`
- `vllm.model_executor.kernels.linear -> AiterFp8BlockScaledMMKernel, ChannelWiseTorchFP8ScaledMMLinearKernel, CutlassFp8BlockScaledMMKernel, CutlassFP8ScaledMMLinearKernel, DeepGemmFp8BlockScaledMMKernel, FlashInferFp8DeepGEMMDynamicBlockScaledKernel, FlashInferFP8ScaledMMLinearKernel, PerTensorTorchFP8ScaledMMLinearKernel, ROCmFP8ScaledMMLinearKernel, RowWiseTorchFP8ScaledMMLinearKernel, TritonFp8BlockScaledMMKernel, _KernelT`
- `vllm.model_executor.layers.layernorm -> RMSNorm, RMSNormGated`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> GroupShape, create_fp8_quant_key`
