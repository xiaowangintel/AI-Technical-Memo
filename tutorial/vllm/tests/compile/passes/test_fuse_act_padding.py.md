# test_fuse_act_padding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_fuse_act_padding.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_fuse_act_padding, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_fuse_act_padding 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-21)
```python
import pytest
import torch

import vllm.config
from tests.compile.backend import TestBackend
from vllm._aiter_ops import rocm_aiter_ops
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
from vllm.model_executor.layers.utils import rocm_unquantized_gemm
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.compile.backend; and vLLM components like vllm.config, vllm._aiter_ops, vllm.compilation.passes.utility.noop_elimination, vllm.compilation.passes.utility.post_cleanup.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.compile.backend；vLLM 内部组件，例如 vllm.config、vllm._aiter_ops、vllm.compilation.passes.utility.noop_elimination、vllm.compilation.passes.utility.post_cleanup。

### Class `TestModel` (lines 24-24)
```python
class TestModel(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestModel.__init__` (lines 25-41)
```python
    def __init__(
        self,
        num_layers: int,
        hidden_size: int,
        num_local_experts: int,
        x_pad_to_multiple: int,
    ):
        super().__init__()
        self.num_layers = num_layers
        self.hidden_size = hidden_size
        self.x_pad_to_multiple = x_pad_to_multiple
        self.pad_dim = x_pad_to_multiple - (hidden_size % x_pad_to_multiple)

        self.norm = [RMSNorm(hidden_size, eps=1e-5) for _ in range(num_layers)]
        self.router = [
            torch.nn.Linear(hidden_size, num_local_experts) for _ in range(4)
        ]
```
**EN:** This method implements the initialization for `TestModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestModel.forward` (lines 43-58)
```python
    def forward(self, x):
        # avoid having graph input be an arg to a pattern directly
        x = resid = torch.relu(x)
        all_router_logits = []
        for layer in range(self.num_layers):
            x = x[:, : self.hidden_size]
            x, resid = self.norm[layer](x, resid)
            router_logits = rocm_unquantized_gemm(
                self, x, self.router[layer].weight, self.router[layer].bias
            )
            x = torch.nn.functional.pad(
                x, (0, self.pad_dim), mode="constant", value=0.0
            )
            all_router_logits.append(router_logits)

        return x, resid, *all_router_logits
```
**EN:** This method on `TestModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestModel.ops_in_model_before` (lines 60-64)
```python
    def ops_in_model_before(self):
        return [
            torch.ops.vllm_ir.fused_add_rms_norm,
            torch.ops.aten.constant_pad_nd,
        ]
```
**EN:** This method on `TestModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestModel.ops_in_model_after` (lines 66-67)
```python
    def ops_in_model_after(self):
        return [rocm_aiter_ops.get_triton_add_rmsnorm_pad_op()]
```
**EN:** This method on `TestModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_fuse_act_padding` (lines 70-128)
```python
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("num_layers", [3])
@pytest.mark.parametrize("hidden_size", [2880])
@pytest.mark.parametrize("num_local_experts", [128])
@pytest.mark.parametrize("x_pad_to_multiple", [256])
@pytest.mark.skip(
    reason="Skipping for now because of the accuracy issue. See: https://github.com/ROCm/aiter/issues/2614"
)
def test_fuse_act_padding(
    dtype: torch.dtype,
    num_layers: int,
    hidden_size: int,
    num_local_experts: int,
    x_pad_to_multiple: int,
    monkeypatch: pytest.MonkeyPatch,
):
    vllm_config = VllmConfig(
        model_config=ModelConfig(dtype=dtype),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=["+rms_norm"],
            pass_config=PassConfig(fuse_act_padding=True, eliminate_noops=True),
        ),
    )

    with vllm.config.set_current_vllm_config(vllm_config), monkeypatch.context() as m:
        from vllm.compilation.passes.fusion.rocm_aiter_fusion import (
            RocmAiterTritonAddRMSNormPadFusionPass,
        )

        torch.set_default_device("cuda")
        torch.set_default_dtype(dtype)
        torch.manual_seed(1)

        m.setenv("VLLM_ROCM_USE_AITER", "1")
        rocm_aiter_ops.refresh_env_variables()

        fusion_pass = RocmAiterTritonAddRMSNormPadFusionPass(vllm_config)
        passes = [
            NoOpEliminationPass(vllm_config),
            fusion_pass,
            PostCleanupPass(vllm_config),
        ]
        backend = TestBackend(*passes)
        model = TestModel(num_layers, hidden_size, num_local_experts, x_pad_to_multiple)

        x = torch.rand(1, hidden_size)
        torch._dynamo.mark_dynamic(x, 0)
        outputs_unfused = model(x)

        model_fused = torch.compile(model, backend=backend)
        outputs_fused = model_fused(x)

        torch.testing.assert_close(outputs_unfused, outputs_fused)

        assert fusion_pass.matched_count == num_layers

        backend.check_before_ops(model.ops_in_model_before())
        backend.check_after_ops(model.ops_in_model_after())
```
**EN:** This pytest case verifies fuse act padding. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dtype, num_layers, hidden_size, num_local_experts. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 fuse act padding 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 dtype、num_layers、hidden_size、num_local_experts 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.config`
- `tests.compile.backend -> TestBackend`
- `vllm._aiter_ops -> rocm_aiter_ops`
- `vllm.compilation.passes.utility.noop_elimination -> NoOpEliminationPass`
- `vllm.compilation.passes.utility.post_cleanup -> PostCleanupPass`
- `vllm.config -> CompilationConfig, CompilationMode, ModelConfig, PassConfig, VllmConfig`
- `vllm.model_executor.layers.layernorm -> RMSNorm`
- `vllm.model_executor.layers.utils -> rocm_unquantized_gemm`
