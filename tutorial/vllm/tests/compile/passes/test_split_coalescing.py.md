# test_split_coalescing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_split_coalescing.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_split_coalescing, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_split_coalescing 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-11)
```python
import pytest
import torch

import vllm
from tests.compile.backend import TestBackend
from vllm.compilation.passes.utility.split_coalescing import SplitCoalescingPass
from vllm.config import CompilationConfig, CompilationMode, PassConfig, VllmConfig
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.compile.backend; and vLLM components like vllm, vllm.compilation.passes.utility.split_coalescing, vllm.config, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.compile.backend；vLLM 内部组件，例如 vllm、vllm.compilation.passes.utility.split_coalescing、vllm.config、vllm.platforms。

### Constants and module state (lines 13-13)
```python
DEVICE_TYPE = current_platform.device_type
```
**EN:** This block centralizes shared constants and parameter grids, including DEVICE_TYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DEVICE_TYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `SplitCoalescingModel` (lines 16-19)
```python
class SplitCoalescingModel(torch.nn.Module):
    """Model with 3 separate split_with_sizes calls on the same input,
    simulating the B200+FP8 graph where CSE fails to merge them."""
```
**EN:** This helper class groups the state and behavior needed for SplitCoalescingModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 SplitCoalescingModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `SplitCoalescingModel.__init__` (lines 20-23)
```python
    def __init__(self, q_size: int, kv_size: int) -> None:
        super().__init__()
        self.q_size = q_size
        self.kv_size = kv_size
```
**EN:** This method implements the initialization for `SplitCoalescingModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `SplitCoalescingModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `SplitCoalescingModel.forward` (lines 25-29)
```python
    def forward(self, qkv: torch.Tensor):
        q, _, _ = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        _, k, _ = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        _, _, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        return q + 1, k + 2, v + 3
```
**EN:** This method on `SplitCoalescingModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `SplitCoalescingModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_split_coalescing` (lines 32-65)
```python
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
def test_split_coalescing(dtype):
    torch.set_default_device(DEVICE_TYPE)
    torch.set_default_dtype(dtype)
    torch.manual_seed(0)

    q_size, kv_size = 2048, 512

    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            pass_config=PassConfig(),
        )
    )
    with vllm.config.set_current_vllm_config(vllm_config):
        coalesce_pass = SplitCoalescingPass(vllm_config)
        backend = TestBackend(coalesce_pass)

        model = SplitCoalescingModel(q_size, kv_size)

        T = 5
        qkv = torch.randn(T, q_size + 2 * kv_size)
        torch._dynamo.mark_dynamic(qkv, 0)

        result_eager = model(qkv)

        model_compiled = torch.compile(model, backend=backend)
        result_compiled = model_compiled(qkv)

        ATOL, RTOL = (2e-3, 2e-3)
        for eager, compiled in zip(result_eager, result_compiled):
            torch.testing.assert_close(eager, compiled, atol=ATOL, rtol=RTOL)

        assert backend.op_count(torch.ops.aten.split_with_sizes.default) == 1
```
**EN:** This pytest case verifies split coalescing. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as dtype. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 split coalescing 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 dtype 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm`
- `tests.compile.backend -> TestBackend`
- `vllm.compilation.passes.utility.split_coalescing -> SplitCoalescingPass`
- `vllm.config -> CompilationConfig, CompilationMode, PassConfig, VllmConfig`
- `vllm.platforms -> current_platform`
