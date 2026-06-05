# test_fuse_mla_dual_rms_norm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_fuse_mla_dual_rms_norm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_fuse_mla_dual_rms_norm, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_fuse_mla_dual_rms_norm 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-8)
```python
"""
Unit test for the MLADualRMSNormFusionPass.

The pass fuses paired q/kv RMS norms in MLA attention into a single
fused_mla_dual_rms_norm op backed by AITER's fused_qk_rmsnorm kernel.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 10-25)
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
from vllm.model_executor.layers.layernorm import RMSNorm
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.compile.backend; and vLLM components like vllm.config, vllm._aiter_ops, vllm.compilation.passes.utility.noop_elimination, vllm.compilation.passes.utility.post_cleanup.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.compile.backend；vLLM 内部组件，例如 vllm.config、vllm._aiter_ops、vllm.compilation.passes.utility.noop_elimination、vllm.compilation.passes.utility.post_cleanup。

### Constants and module state (lines 28-31)
```python
Q_DIM = 1536
KV_C_DIM = 512
K_PE_DIM = 64
EPS = 1e-6
```
**EN:** This block centralizes shared constants and parameter grids, including Q_DIM, KV_C_DIM, K_PE_DIM, EPS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 Q_DIM、KV_C_DIM、K_PE_DIM、EPS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `MLADualRMSNormTestModel` (lines 34-43)
```python
class MLADualRMSNormTestModel(torch.nn.Module):
    """
    Minimal model reproducing the MLA dual RMS norm pattern:
        linear -> split([q_dim, kv_dim])
            +-- q_c (getitem 0) -> rms_norm(q_w, eps) -> linear
            +-- kv_lora (getitem 1) -> split([kv_c_dim, k_pe_dim])
                    +-- kv_c (getitem 0) -> rms_norm(kv_w, eps)
                    +-- k_pe
    """
```
**EN:** This helper class groups the state and behavior needed for MLADualRMSNormTestModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 MLADualRMSNormTestModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `MLADualRMSNormTestModel.__init__` (lines 44-61)
```python
    def __init__(
        self,
        hidden_size: int,
        q_dim: int = Q_DIM,
        kv_c_dim: int = KV_C_DIM,
        k_pe_dim: int = K_PE_DIM,
        eps: float = EPS,
    ):
        super().__init__()
        self.q_dim = q_dim
        self.kv_dim = kv_c_dim + k_pe_dim
        self.kv_c_dim = kv_c_dim
        self.k_pe_dim = k_pe_dim

        self.proj = torch.nn.Linear(hidden_size, q_dim + self.kv_dim, bias=False)
        self.q_norm = RMSNorm(q_dim, eps=eps)
        self.kv_norm = RMSNorm(kv_c_dim, eps=eps)
        self.q_b_proj = torch.nn.Linear(q_dim, hidden_size, bias=False)
```
**EN:** This method implements the initialization for `MLADualRMSNormTestModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `MLADualRMSNormTestModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MLADualRMSNormTestModel.forward` (lines 63-78)
```python
    def forward(
        self, x: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        # Avoid graph input being a direct arg to a matched pattern node
        x = torch.relu(x)

        projected = self.proj(x)

        q_c, kv_lora = projected.split([self.q_dim, self.kv_dim], dim=-1)
        kv_c, k_pe = kv_lora.split([self.kv_c_dim, self.k_pe_dim], dim=-1)

        q_normed = self.q_norm(q_c)
        kv_normed = self.kv_norm(kv_c)

        q_out = self.q_b_proj(q_normed)
        return q_out, kv_normed, k_pe
```
**EN:** This method on `MLADualRMSNormTestModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MLADualRMSNormTestModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MLADualRMSNormTestModel.ops_in_model_before` (lines 80-81)
```python
    def ops_in_model_before(self):
        return [torch.ops.vllm_ir.rms_norm.default]
```
**EN:** This method on `MLADualRMSNormTestModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MLADualRMSNormTestModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `MLADualRMSNormTestModel.ops_in_model_after` (lines 83-84)
```python
    def ops_in_model_after(self):
        return [torch.ops.vllm.fused_mla_dual_rms_norm.default]
```
**EN:** This method on `MLADualRMSNormTestModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `MLADualRMSNormTestModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_fuse_mla_dual_rms_norm` (lines 87-148)
```python
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("hidden_size", [7168])
@pytest.mark.skipif(
    not is_aiter_found_and_supported(),
    reason="Only test on ROCm with AITER installed and supported",
)
def test_fuse_mla_dual_rms_norm(
    dtype: torch.dtype,
    hidden_size: int,
    monkeypatch: pytest.MonkeyPatch,
):
    torch._dynamo.reset()

    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=dtype),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=["+rms_norm"],
            pass_config=PassConfig(
                fuse_mla_dual_rms_norm=True,
                eliminate_noops=True,
            ),
        ),
    )

    with vllm.config.set_current_vllm_config(vllm_config), monkeypatch.context() as m:
        from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
            MLADualRMSNormFusionPass,
        )

        torch.set_default_device("cuda")
        torch.set_default_dtype(dtype)
        torch.manual_seed(42)

        m.setenv("VLLM_ROCM_USE_AITER", "1")
        rocm_aiter_ops.refresh_env_variables()

        fusion_pass = MLADualRMSNormFusionPass(vllm_config)
        passes = [
            NoOpEliminationPass(vllm_config),
            fusion_pass,
            PostCleanupPass(vllm_config),
        ]
        backend = TestBackend(*passes)
        model = MLADualRMSNormTestModel(hidden_size)

        x = torch.randn(1, hidden_size)
        torch._dynamo.mark_dynamic(x, 0)

        outputs_unfused = model(x)

        model_fused = torch.compile(model, backend=backend)
        outputs_fused = model_fused(x)

        torch.testing.assert_close(outputs_unfused, outputs_fused, atol=1e-2, rtol=1e-2)

        assert fusion_pass.matched_count == 1, (
            f"Expected 1 fused pair, got {fusion_pass.matched_count}"
        )

        backend.check_before_ops(model.ops_in_model_before())
        backend.check_after_ops(model.ops_in_model_after())
```
**EN:** This pytest case verifies fuse mla dual rms norm. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dtype, hidden_size, monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 fuse mla dual rms norm 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 dtype、hidden_size、monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

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
- `vllm.model_executor.layers.layernorm -> RMSNorm`
