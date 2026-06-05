# test_silu_mul_quant_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_silu_mul_quant_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_silu_mul_quant_fusion, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_silu_mul_quant_fusion 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-47)
```python
import itertools
from functools import partial

import pytest
import torch

import vllm.envs as envs
from tests.compile.backend import TestBackend
from tests.kernels.quantization.nvfp4_utils import quant_nvfp4_tensor
from tests.utils import TestFP8Layer
from vllm._aiter_ops import IS_AITER_FOUND, rocm_aiter_ops
from vllm._custom_ops import cutlass_scaled_fp4_mm, scaled_fp4_quant
from vllm.compilation.passes.fusion.act_quant_fusion import (
    FUSED_OPS,
    SILU_MUL_OP,
    ActivationQuantFusionPass,
)
from vllm.compilation.passes.fusion.rms_quant_fusion import QUANT_OPS
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    PassConfig,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.model_executor.kernels.linear import (
    CutlassFP8ScaledMMLinearKernel,
    FlashInferFP8ScaledMMLinearKernel,
    FP8ScaledMMLinearKernel,
    PerTensorTorchFP8ScaledMMLinearKernel,
    ROCmFP8ScaledMMLinearKernel,
)
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    create_fp8_quant_key,
    kFp8Dynamic128Sym,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
)
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import is_deep_gemm_supported
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as itertools, functools, pytest, torch; shared test helpers from tests.compile.backend, tests.kernels.quantization.nvfp4_utils, tests.utils; and vLLM components like vllm.envs, vllm._aiter_ops, vllm._custom_ops, vllm.compilation.passes.fusion.act_quant_fusion.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 itertools、functools、pytest、torch；共享测试辅助模块，例如 tests.compile.backend、tests.kernels.quantization.nvfp4_utils、tests.utils；vLLM 内部组件，例如 vllm.envs、vllm._aiter_ops、vllm._custom_ops、vllm.compilation.passes.fusion.act_quant_fusion。

### Constants and module state (lines 49-50)
```python
FP8_DTYPE = current_platform.fp8_dtype()
FP4_DTYPE = torch.uint8
```
**EN:** This block centralizes shared constants and parameter grids, including FP8_DTYPE, FP4_DTYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FP8_DTYPE、FP4_DTYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `is_nvfp4_supported` (lines 53-54)
```python
def is_nvfp4_supported():
    return current_platform.has_device_capability(100)
```
**EN:** This helper function implements the shared logic for is nvfp4 supported. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 is nvfp4 supported 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestSiluMulFp8QuantModel` (lines 57-59)
```python
class TestSiluMulFp8QuantModel(torch.nn.Module):
    quant_key = kFp8StaticTensorSym
```
**EN:** This helper class groups the state and behavior needed for TestSiluMulFp8QuantModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestSiluMulFp8QuantModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestSiluMulFp8QuantModel.__init__` (lines 60-79)
```python
    def __init__(
        self,
        hidden_size: int,
        force_kernel: FP8ScaledMMLinearKernel,
        dtype: torch.dtype,
        **kwargs,
    ):
        super().__init__()
        self.silu_and_mul = SiluAndMul()

        self.fp8_linear = TestFP8Layer(
            weight_shape=(hidden_size, hidden_size),
            activation_quant_key=self.quant_key,
            weight_quant_key=self.quant_key,
            force_kernel=force_kernel,
            input_dtype=dtype,
        )

        self.enable_silu_mul_custom_op = self.silu_and_mul.enabled()
        self.enable_quant_fp8_custom_op = self.fp8_linear.is_quant_fp8_enabled()
```
**EN:** This method implements the initialization for `TestSiluMulFp8QuantModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestSiluMulFp8QuantModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulFp8QuantModel.forward` (lines 81-84)
```python
    def forward(self, x):
        y = self.silu_and_mul(x)
        x2 = self.fp8_linear(y)
        return x2
```
**EN:** This method on `TestSiluMulFp8QuantModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulFp8QuantModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulFp8QuantModel.ops_in_model_before` (lines 86-94)
```python
    def ops_in_model_before(self):
        return [
            SILU_MUL_OP if self.enable_silu_mul_custom_op else torch.ops.aten.mul,
            (
                QUANT_OPS[kFp8StaticTensorSym]
                if self.enable_quant_fp8_custom_op
                else torch.ops.aten.reciprocal
            ),
        ]
```
**EN:** This method on `TestSiluMulFp8QuantModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulFp8QuantModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulFp8QuantModel.ops_in_model_after` (lines 96-97)
```python
    def ops_in_model_after(self):
        return [FUSED_OPS[kFp8StaticTensorSym]]
```
**EN:** This method on `TestSiluMulFp8QuantModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulFp8QuantModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestSiluMulNvfp4QuantModel` (lines 100-100)
```python
class TestSiluMulNvfp4QuantModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestSiluMulNvfp4QuantModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestSiluMulNvfp4QuantModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestSiluMulNvfp4QuantModel.__init__` (lines 101-119)
```python
    def __init__(self, hidden_size: int, x: torch.Tensor, **kwargs):
        super().__init__()
        from vllm.compilation.passes.fusion.act_quant_fusion import (
            silu_and_mul_nvfp4_quant_supported,
        )

        assert silu_and_mul_nvfp4_quant_supported

        self.silu_and_mul = SiluAndMul()
        self.enable_silu_mul_custom_op = self.silu_and_mul.enabled()

        # create nvfp4 weight
        w = torch.rand((hidden_size, hidden_size))
        self.w, self.w_block_scale, self.w_global_scale = quant_nvfp4_tensor(w)

        # get global scale offline
        _, _, self.y_global_scale = quant_nvfp4_tensor(self.silu_and_mul(x))

        self.alpha = 1.0 / (self.w_global_scale * self.y_global_scale)
```
**EN:** This method implements the initialization for `TestSiluMulNvfp4QuantModel`. assertions at the end lock in the intended behavior or graph shape.
**CN:** 这个方法实现了 `TestSiluMulNvfp4QuantModel` 的初始化逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestSiluMulNvfp4QuantModel.forward` (lines 121-132)
```python
    def forward(self, x):
        y = self.silu_and_mul(x)
        y_quant, y_block_scale = scaled_fp4_quant(y, self.y_global_scale)
        out = cutlass_scaled_fp4_mm(
            a=y_quant,
            b=self.w,
            block_scale_a=y_block_scale,
            block_scale_b=self.w_block_scale,
            alpha=self.alpha,
            out_dtype=y.dtype,
        )
        return out
```
**EN:** This method on `TestSiluMulNvfp4QuantModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulNvfp4QuantModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulNvfp4QuantModel.ops_in_model_before` (lines 134-138)
```python
    def ops_in_model_before(self):
        return [
            SILU_MUL_OP if self.enable_silu_mul_custom_op else torch.ops.aten.mul,
            QUANT_OPS[kNvfp4Dynamic],
        ]
```
**EN:** This method on `TestSiluMulNvfp4QuantModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulNvfp4QuantModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulNvfp4QuantModel.ops_in_model_after` (lines 140-141)
```python
    def ops_in_model_after(self):
        return [FUSED_OPS[kNvfp4Dynamic]]
```
**EN:** This method on `TestSiluMulNvfp4QuantModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulNvfp4QuantModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestSiluMulGroupFp8QuantModel` (lines 144-146)
```python
class TestSiluMulGroupFp8QuantModel(torch.nn.Module):
    act_quant_key = kFp8Dynamic128Sym
```
**EN:** This helper class groups the state and behavior needed for TestSiluMulGroupFp8QuantModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestSiluMulGroupFp8QuantModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestSiluMulGroupFp8QuantModel.__init__` (lines 147-168)
```python
    def __init__(self, hidden_size: int, dtype: torch.dtype, **kwargs):
        super().__init__()
        self.silu_and_mul = SiluAndMul()
        self.weight_quant_key = create_fp8_quant_key(
            static=True, group_shape=GroupShape(hidden_size, hidden_size)
        )

        self.w8a8_block_fp8_linear = TestFP8Layer(
            weight_shape=(hidden_size, hidden_size),
            weight_quant_key=self.weight_quant_key,
            activation_quant_key=self.act_quant_key,
            input_dtype=dtype,
        )

        if not current_platform.is_fp8_fnuz():
            kernel = self.w8a8_block_fp8_linear.kernel
            orig_quant = kernel.quant_fp8
            kernel.quant_fp8 = lambda *a, use_triton=False, **kw: orig_quant(
                *a, use_triton=True, **kw
            )

        self.enable_silu_mul_custom_op = self.silu_and_mul.enabled()
```
**EN:** This method implements the initialization for `TestSiluMulGroupFp8QuantModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestSiluMulGroupFp8QuantModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulGroupFp8QuantModel.forward` (lines 170-173)
```python
    def forward(self, x):
        y = self.silu_and_mul(x)
        x2 = self.w8a8_block_fp8_linear(y)
        return x2
```
**EN:** This method on `TestSiluMulGroupFp8QuantModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulGroupFp8QuantModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulGroupFp8QuantModel.ops_in_model_before` (lines 175-181)
```python
    def ops_in_model_before(self):
        return [
            SILU_MUL_OP if self.enable_silu_mul_custom_op else torch.ops.aten.mul,
            rocm_aiter_ops.get_group_quant_op()
            if current_platform.is_fp8_fnuz()
            else torch.ops.vllm.triton_per_token_group_quant_fp8.default,
        ]
```
**EN:** This method on `TestSiluMulGroupFp8QuantModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulGroupFp8QuantModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulGroupFp8QuantModel.ops_in_model_after` (lines 183-184)
```python
    def ops_in_model_after(self):
        return [torch.ops.vllm.rocm_aiter_act_mul_and_fp8_group_quant]
```
**EN:** This method on `TestSiluMulGroupFp8QuantModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulGroupFp8QuantModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestSiluMulBlockQuantModel` (lines 187-189)
```python
class TestSiluMulBlockQuantModel(torch.nn.Module):
    quant_key = kFp8Dynamic128Sym
```
**EN:** This helper class groups the state and behavior needed for TestSiluMulBlockQuantModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestSiluMulBlockQuantModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestSiluMulBlockQuantModel.__init__` (lines 190-202)
```python
    def __init__(self, hidden_size: int, is_scale_transposed: bool = False, **kwargs):
        super().__init__()
        self.silu_and_mul = SiluAndMul()
        self.is_scale_transposed = is_scale_transposed
        self.quant_fp8 = QuantFP8(
            static=False,
            group_shape=GroupShape(1, 128),
            column_major_scales=is_scale_transposed,
            compile_native=False,
        )

        self.enable_silu_mul_custom_op = self.silu_and_mul.enabled()
        self.enable_quant_fp8_custom_op = self.quant_fp8.enabled()
```
**EN:** This method implements the initialization for `TestSiluMulBlockQuantModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestSiluMulBlockQuantModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulBlockQuantModel.forward` (lines 204-210)
```python
    def forward(self, x):
        y = self.silu_and_mul(x)
        out, scale = self.quant_fp8(y)
        group_size = self.quant_key.scale.group_shape[1]
        scale_expanded = scale.repeat_interleave(group_size, dim=1)
        dequant = out.to(dtype=torch.float32) * scale_expanded
        return (dequant,)
```
**EN:** This method on `TestSiluMulBlockQuantModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulBlockQuantModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulBlockQuantModel.ops_in_model_before` (lines 212-223)
```python
    def ops_in_model_before(self):
        ops = []
        if self.enable_silu_mul_custom_op:
            ops.append(SILU_MUL_OP)
        # When silu custom op is disabled, aten.mul.Tensor also appears
        # in dequant code, so we skip checking it to avoid false positives.
        ops.append(
            QUANT_OPS[self.quant_key]
            if self.enable_quant_fp8_custom_op
            else torch.ops.aten.reciprocal.default
        )
        return ops
```
**EN:** This method on `TestSiluMulBlockQuantModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulBlockQuantModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestSiluMulBlockQuantModel.ops_in_model_after` (lines 225-226)
```python
    def ops_in_model_after(self):
        return [FUSED_OPS[self.quant_key]]
```
**EN:** This method on `TestSiluMulBlockQuantModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestSiluMulBlockQuantModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 229-235)
```python
ROCM_KERNELS = [ROCmFP8ScaledMMLinearKernel, PerTensorTorchFP8ScaledMMLinearKernel]
CUDA_KERNELS = [
    FlashInferFP8ScaledMMLinearKernel,
    CutlassFP8ScaledMMLinearKernel,
    PerTensorTorchFP8ScaledMMLinearKernel,
]
TEST_KERNELS = ROCM_KERNELS if current_platform.is_rocm() else CUDA_KERNELS
```
**EN:** This block centralizes shared constants and parameter grids, including ROCM_KERNELS, CUDA_KERNELS, TEST_KERNELS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 ROCM_KERNELS、CUDA_KERNELS、TEST_KERNELS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_fusion_silu_and_mul_quant` (lines 238-379)
```python
@pytest.mark.parametrize("num_tokens", [32, 64])
@pytest.mark.parametrize("hidden_size", [128, 256])
@pytest.mark.parametrize("dtype", [torch.bfloat16, torch.float16])
@pytest.mark.parametrize("enable_silu_mul_custom_op", [True, False])
@pytest.mark.parametrize(
    "model_class, enable_quant_fp8_custom_op, force_kernel",
    list(itertools.product([TestSiluMulFp8QuantModel], [True, False], TEST_KERNELS))
    + [
        pytest.param(
            TestSiluMulNvfp4QuantModel,
            False,
            None,
            marks=pytest.mark.skipif(
                not current_platform.is_cuda(), reason="CUDA only"
            ),
        ),
        # GroupFP8Quant fusion only works with AITER on ROCm.
        # and the enable_quant_fp8_custom_op must be True.
        pytest.param(
            TestSiluMulGroupFp8QuantModel,
            True,
            None,
            marks=pytest.mark.skipif(
                not current_platform.is_rocm(), reason="ROCm only"
            ),
        ),
        # Block quant fusion for per-group FP8 (CUDA only).
        *[
            pytest.param(
                partial(TestSiluMulBlockQuantModel, is_scale_transposed=transposed),
                True,
                None,
                marks=pytest.mark.skipif(
                    not current_platform.is_cuda(), reason="CUDA only"
                ),
                id=f"TestSiluMulBlockQuant-transposed={transposed}",
            )
            for transposed in [False, True]
        ],
    ],
# ... excerpt ...
        model = model_class(
            hidden_size=hidden_size, force_kernel=force_kernel, x=x, dtype=dtype
        )

        # First dimension dynamic
        torch._dynamo.mark_dynamic(x, 0)

        result = model(x)

        model2 = torch.compile(model, backend=backend)
        result2 = model2(x)

        # Check that it gives the same answer
        if isinstance(model, TestSiluMulFp8QuantModel):
            atol, rtol = 1e-3, 1e-3
        elif isinstance(model, TestSiluMulNvfp4QuantModel):
            atol, rtol = 1e-1, 1e-1
        elif isinstance(model, TestSiluMulGroupFp8QuantModel):
            atol, rtol = 5e-2, 5e-2
        elif isinstance(model, TestSiluMulBlockQuantModel):
            if current_platform.is_rocm():
                atol, rtol = 1e-3, 1e-3
            else:
                # CUDA fused kernel computes silu*mul in fp32 while the reference
                # goes through bf16/fp16 storage, so group maxima (and thus scales)
                # can shift by one FP8-e4m3 code (~1/8 relative step).
                atol, rtol = 5e-2, 5e-2

        torch.testing.assert_close(
            result[0].to(dtype=dtype), result2[0].to(dtype=dtype), atol=atol, rtol=rtol
        )

        assert sum([p.matched_count for p in fusion_passes]) == 1

        # In pre-nodes, quant op should be present and fused kernels should not
        backend.check_before_ops(model.ops_in_model_before())

        # In post-nodes, fused kernels should be present and quant op should not
        backend.check_after_ops(model.ops_in_model_after())
```
**EN:** This pytest case verifies fusion silu and mul quant. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, dtype, model_class. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 fusion silu and mul quant 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、dtype、model_class 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `itertools`
- `functools -> partial`
- `pytest`
- `torch`
- `vllm.envs`
- `tests.compile.backend -> TestBackend`
- `tests.kernels.quantization.nvfp4_utils -> quant_nvfp4_tensor`
- `tests.utils -> TestFP8Layer`
- `vllm._aiter_ops -> IS_AITER_FOUND, rocm_aiter_ops`
- `vllm._custom_ops -> cutlass_scaled_fp4_mm, scaled_fp4_quant`
- `vllm.compilation.passes.fusion.act_quant_fusion -> FUSED_OPS, SILU_MUL_OP, ActivationQuantFusionPass`
- `vllm.compilation.passes.fusion.rms_quant_fusion -> QUANT_OPS`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.config -> CompilationConfig, CompilationMode, PassConfig, VllmConfig, set_current_vllm_config`
- `vllm.model_executor.kernels.linear -> CutlassFP8ScaledMMLinearKernel, FlashInferFP8ScaledMMLinearKernel, FP8ScaledMMLinearKernel, PerTensorTorchFP8ScaledMMLinearKernel, ROCmFP8ScaledMMLinearKernel`
