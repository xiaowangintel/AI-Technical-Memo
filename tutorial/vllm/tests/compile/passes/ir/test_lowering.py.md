# test_lowering.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/ir/test_lowering.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / ir / test_lowering, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / ir / test_lowering 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-16)
```python
import pytest
import torch
from torch import nn

import vllm.kernels  # noqa: F401 to register kernels
from vllm import ir
from vllm.compilation.passes.ir.lowering_pass import (
    VllmIRLoweringPass,
)
from vllm.config import get_current_vllm_config
from vllm.ir import ops
from vllm.platforms import current_platform

from ...backend import TestBackend
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from ...backend; and vLLM components like vllm.kernels  # noqa: F401 to register kernels, vllm, vllm.compilation.passes.ir.lowering_pass, vllm.config.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 ...backend；vLLM 内部组件，例如 vllm.kernels  # noqa: F401 to register kernels、vllm、vllm.compilation.passes.ir.lowering_pass、vllm.config。

### Class `Model` (lines 19-19)
```python
class Model(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for Model. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 Model 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `Model.__init__` (lines 20-23)
```python
    def __init__(self, hidden_size=16, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.hidden_size = hidden_size
        self.weight = torch.ones(hidden_size, dtype=torch.bfloat16)
```
**EN:** This method implements the initialization for `Model`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `Model` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `Model.forward` (lines 25-34)
```python
    def forward(self, x):
        x1 = x + 4.0
        x2 = ops.rms_norm(x1, self.weight, 1e-5)
        x3 = x2 * 5.0
        # no weight
        x4 = ops.rms_norm(x3, None, 1e-5)
        x5 = x4 / 2.0
        # dispatch to native due to variance_size parameter
        x6 = ops.rms_norm(x5, self.weight, 1e-5, self.hidden_size // 2)
        return x6 + 3.0
```
**EN:** This method on `Model` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `Model` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_lowering_rms_norm` (lines 37-69)
```python
@pytest.mark.parametrize("rms_provider", ops.rms_norm.supported_providers())
def test_lowering_rms_norm(rms_provider, default_vllm_config):
    torch.set_default_device(current_platform.device_type)

    lowering_pass = VllmIRLoweringPass(get_current_vllm_config())
    backend = TestBackend(lowering_pass)
    backend_unlowered = TestBackend()

    model = Model()
    x = torch.randn(8, 16, dtype=torch.bfloat16)
    with (
        ops.rms_norm.set_priority([rms_provider, "native"]),
        ir.enable_torch_wrap(True),
    ):
        compiled_model = torch.compile(model, backend=backend, fullgraph=True)
        compiled_unlowered_model = torch.compile(
            model, backend=backend_unlowered, fullgraph=True
        )
        output = compiled_model(x)
        output_unlowered = compiled_unlowered_model(x)

    selected = lowering_pass.selected_impls["rms_norm"]
    assert len(selected) == 3
    assert selected["rms_norm"] == rms_provider
    assert selected["rms_norm_1"] == rms_provider
    assert selected["rms_norm_2"] == "native"

    # Compiled function guards on global value, avoid recompilation
    with ir.enable_torch_wrap(True):
        output2 = compiled_model(x)

    torch.testing.assert_close(output_unlowered, output)
    torch.testing.assert_close(output_unlowered, output2)
```
**EN:** This pytest case verifies lowering rms norm. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as rms_provider, default_vllm_config. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 lowering rms norm 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 rms_provider、default_vllm_config 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch -> nn`
- `vllm.kernels`
- `vllm -> ir`
- `vllm.compilation.passes.ir.lowering_pass -> VllmIRLoweringPass`
- `vllm.config -> get_current_vllm_config`
- `vllm.ir -> ops`
- `vllm.platforms -> current_platform`
- `...backend -> TestBackend`
