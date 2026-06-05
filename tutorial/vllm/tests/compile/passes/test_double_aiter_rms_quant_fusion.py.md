# test_double_aiter_rms_quant_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_double_aiter_rms_quant_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_double_aiter_rms_quant_fusion, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_double_aiter_rms_quant_fusion 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-18)
```python
"""
Unit tests for the DoubleQuant fan-out variants registered by
``RocmAiterRMSNormQuantFusionPass``.

Both variants target a 1-to-2 fan-out where one ``rms_norm`` output feeds
two distinct ``rocm_aiter_group_fp8_quant`` consumers and rewrite it into
two independent fused ``rms_norm + group_fp8_quant`` ops:

* ``DoubleAiterRMSFp8GroupQuantPattern`` matches the un-viewed shape
  (e.g. Kimi-K2.5 / DSR1).
* ``DoubleAiterRMSFp8GroupQuantViewPattern`` (this PR) is the view-tolerant
  sibling that additionally matches the
  ``rms_norm -> view -> group_fp8_quant`` shape that DSv3.2's MLA indexer
  q_c norm exposes through ``Fp8BlockScaledMMLinearKernel.apply_weights``'s
  2D-flatten boilerplate.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 20-34)
```python
import pytest
import torch

import vllm.config
from tests.compile.backend import TestBackend
from vllm._aiter_ops import is_aiter_found_and_supported, rocm_aiter_ops
from vllm.compilation.passes.utility.noop_elimination import NoOpEliminationPass
from vllm.compilation.passes.utility.post_cleanup import PostCleanupPass
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    ModelConfig,
    PassConfig,
    VllmConfig,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.compile.backend; and vLLM components like vllm.config, vllm._aiter_ops, vllm.compilation.passes.utility.noop_elimination, vllm.compilation.passes.utility.post_cleanup.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.compile.backend；vLLM 内部组件，例如 vllm.config、vllm._aiter_ops、vllm.compilation.passes.utility.noop_elimination、vllm.compilation.passes.utility.post_cleanup。

### Constants and module state (lines 36-38)
```python
EPS = 1e-5
HIDDEN_SIZE = 256
GROUP_SIZE = 128
```
**EN:** This block centralizes shared constants and parameter grids, including EPS, HIDDEN_SIZE, GROUP_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 EPS、HIDDEN_SIZE、GROUP_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `_NoViewDoubleQuantModel` (lines 41-43)
```python
class _NoViewDoubleQuantModel(torch.nn.Module):
    """``rms_norm -> 2x group_fp8_quant`` fan-out (Kimi-K2.5 / DSR1 shape)."""
```
**EN:** This helper class groups the state and behavior needed for NoViewDoubleQuantModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 NoViewDoubleQuantModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `_NoViewDoubleQuantModel.__init__` (lines 44-46)
```python
    def __init__(self) -> None:
        super().__init__()
        self.weight = torch.nn.Parameter(torch.ones(HIDDEN_SIZE, dtype=torch.bfloat16))
```
**EN:** This method implements the initialization for `_NoViewDoubleQuantModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `_NoViewDoubleQuantModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `_NoViewDoubleQuantModel.forward` (lines 48-56)
```python
    def forward(
        self, x: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
        # avoid graph input being a direct arg to a matched pattern node
        x = torch.relu(x)
        rms = torch.ops.vllm_ir.rms_norm(x, self.weight, EPS)
        q1, s1 = torch.ops.vllm.rocm_aiter_group_fp8_quant.default(rms, GROUP_SIZE)
        q2, s2 = torch.ops.vllm.rocm_aiter_group_fp8_quant.default(rms, GROUP_SIZE)
        return q1, s1, q2, s2
```
**EN:** This method on `_NoViewDoubleQuantModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `_NoViewDoubleQuantModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `_ViewDoubleQuantModel` (lines 59-65)
```python
class _ViewDoubleQuantModel(torch.nn.Module):
    """``rms_norm -> view -> 2x group_fp8_quant`` fan-out (DSv3.2 shape).

    Reproduces the FX-graph shape produced by ``Fp8BlockScaledMMLinearKernel``'s
    2D-flatten before the FP8 group quant op.
    """
```
**EN:** This helper class groups the state and behavior needed for ViewDoubleQuantModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 ViewDoubleQuantModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `_ViewDoubleQuantModel.__init__` (lines 66-68)
```python
    def __init__(self) -> None:
        super().__init__()
        self.weight = torch.nn.Parameter(torch.ones(HIDDEN_SIZE, dtype=torch.bfloat16))
```
**EN:** This method implements the initialization for `_ViewDoubleQuantModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `_ViewDoubleQuantModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `_ViewDoubleQuantModel.forward` (lines 70-78)
```python
    def forward(
        self, x: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
        x = torch.relu(x)
        rms = torch.ops.vllm_ir.rms_norm(x, self.weight, EPS)
        view = rms.view(-1, rms.shape[-1])
        q1, s1 = torch.ops.vllm.rocm_aiter_group_fp8_quant.default(view, GROUP_SIZE)
        q2, s2 = torch.ops.vllm.rocm_aiter_group_fp8_quant.default(view, GROUP_SIZE)
        return q1, s1, q2, s2
```
**EN:** This method on `_ViewDoubleQuantModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `_ViewDoubleQuantModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_double_aiter_rms_fp8_group_quant_fusion` (lines 81-166)
```python
@pytest.mark.parametrize(
    "model_cls",
    [_NoViewDoubleQuantModel, _ViewDoubleQuantModel],
    ids=["no_view", "with_view"],
)
@pytest.mark.skipif(
    not is_aiter_found_and_supported(),
    reason="Only test on ROCm with AITER installed and supported",
)
def test_double_aiter_rms_fp8_group_quant_fusion(
    model_cls: type[torch.nn.Module],
    monkeypatch: pytest.MonkeyPatch,
) -> None:
    """
    Both fan-out shapes (with and without an intermediate view) must fuse
    into ``rocm_aiter_rmsnorm_fp8_group_quant``: the no-view shape via
    ``DoubleAiterRMSFp8GroupQuantPattern`` and the viewed shape via the
    new ``DoubleAiterRMSFp8GroupQuantViewPattern`` sibling.

    A failure on the ``with_view`` parametrization is a regression on the
    DSv3.2 q_c norm path that this PR's view-tolerant pattern is intended
    to cover.
    """
    torch._dynamo.reset()

    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=torch.bfloat16),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=["+rms_norm", "+quant_fp8"],
            pass_config=PassConfig(
                fuse_norm_quant=True,
                eliminate_noops=True,
            ),
        ),
    )

    with vllm.config.set_current_vllm_config(vllm_config), monkeypatch.context() as m:
        from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
            RocmAiterRMSNormQuantFusionPass,
# ... excerpt ...
        rocm_aiter_ops.refresh_env_variables()

        fusion_pass = RocmAiterRMSNormQuantFusionPass(vllm_config)
        passes = [
            NoOpEliminationPass(vllm_config),
            fusion_pass,
            PostCleanupPass(vllm_config),
        ]
        backend = TestBackend(*passes)
        model = model_cls()

        x = torch.randn(8, HIDDEN_SIZE)
        torch._dynamo.mark_dynamic(x, 0)

        outputs_unfused = model(x)
        model_fused = torch.compile(model, backend=backend)
        outputs_fused = model_fused(x)

        # Both consumers must be rewritten into the fused op (one
        # ``register_replacement`` rewrite covers the whole 1-to-2 fan-out).
        assert fusion_pass.matched_count == 1, (
            f"Expected the {model_cls.__name__} fan-out to fuse via the "
            f"DoubleQuant pattern (matched_count == 1), got "
            f"{fusion_pass.matched_count}"
        )

        fused_op = rocm_aiter_ops.get_rmsnorm_group_fused_quant_op()
        backend.check_after_ops([fused_op])

        # Numerical parity sanity-check: the fused pair must match the
        # unfused pair on FP8 outputs (exact byte-equality is the goal,
        # but allow a tiny tolerance for any residual numeric noise).
        for fused_t, unfused_t in zip(outputs_fused, outputs_unfused):
            torch.testing.assert_close(
                fused_t.to(torch.float32),
                unfused_t.to(torch.float32),
                atol=1e-2,
                rtol=1e-2,
            )
```
**EN:** This pytest case verifies double aiter rms FP8 group quant fusion. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as model_cls, monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 double aiter rms FP8 group quant fusion 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 model_cls、monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.config`
- `tests.compile.backend -> TestBackend`
- `vllm._aiter_ops -> is_aiter_found_and_supported, rocm_aiter_ops`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.config -> CompilationConfig, CompilationMode, ModelConfig, PassConfig, VllmConfig`
