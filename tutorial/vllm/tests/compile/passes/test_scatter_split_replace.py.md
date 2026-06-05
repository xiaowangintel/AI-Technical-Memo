# test_scatter_split_replace.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/passes/test_scatter_split_replace.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / passes / test_scatter_split_replace, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / passes / test_scatter_split_replace 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-16)
```python
import pytest
import torch
import torch.nn as nn

import vllm
from tests.compile.backend import TestBackend
from vllm.compilation.passes.utility.scatter_split_replace import (
    ScatterSplitReplacementPass,
)
from vllm.compilation.passes.utility.split_coalescing import SplitCoalescingPass
from vllm.config import CompilationConfig, CompilationMode, VllmConfig
from vllm.model_executor.layers.rotary_embedding import RotaryEmbedding
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch, torch.nn; shared test helpers from tests.compile.backend; and vLLM components like vllm, vllm.compilation.passes.utility.scatter_split_replace, vllm.compilation.passes.utility.split_coalescing, vllm.config.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch、torch.nn；共享测试辅助模块，例如 tests.compile.backend；vLLM 内部组件，例如 vllm、vllm.compilation.passes.utility.scatter_split_replace、vllm.compilation.passes.utility.split_coalescing、vllm.config。

### Constants and module state (lines 18-18)
```python
DEVICE_TYPE = current_platform.device_type
```
**EN:** This block centralizes shared constants and parameter grids, including DEVICE_TYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DEVICE_TYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `ScatterSplitReplacementModel` (lines 21-23)
```python
class ScatterSplitReplacementModel(nn.Module):
    """Model with a rope+getitem+slice_scatter+split_with_sizes sequence."""
```
**EN:** This helper class groups the state and behavior needed for ScatterSplitReplacementModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 ScatterSplitReplacementModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `ScatterSplitReplacementModel.__init__` (lines 24-42)
```python
    def __init__(
        self,
        num_heads: int,
        num_kv_heads: int,
        head_size: int,
        dtype: torch.dtype,
    ):
        super().__init__()
        self.q_size = num_heads * head_size
        self.kv_size = num_kv_heads * head_size

        self.rotary_emb = RotaryEmbedding(
            head_size,
            rotary_dim=head_size,
            max_position_embeddings=4096,
            base=10000,
            is_neox_style=True,
            dtype=dtype,
        )
```
**EN:** This method implements the initialization for `ScatterSplitReplacementModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `ScatterSplitReplacementModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `ScatterSplitReplacementModel.forward` (lines 44-52)
```python
    def forward(self, qkv: torch.Tensor, positions: torch.Tensor):
        # Create copy so inplace ops do not modify the original tensors
        qkv = qkv.clone()
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self.rotary_emb(positions, q, k)
        q = q + 1
        k = k + 2
        v = v + 3
        return q, k, v
```
**EN:** This method on `ScatterSplitReplacementModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `ScatterSplitReplacementModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `ScatterSplitReplacementModel.ops_in_model_before` (lines 54-59)
```python
    def ops_in_model_before(self) -> list[torch._ops.OpOverload]:
        return [
            torch.ops.aten.slice_scatter.default,
            torch.ops.aten.split_with_sizes.default,
            torch.ops.aten.getitem.default,
        ]
```
**EN:** This method on `ScatterSplitReplacementModel` implements ops in model before. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `ScatterSplitReplacementModel` 中的这个方法实现了 ops in model before。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `ScatterSplitReplacementModel.ops_in_model_after` (lines 61-62)
```python
    def ops_in_model_after(self) -> list[torch._ops.OpOverload]:
        return [torch.ops.aten.getitem.default]
```
**EN:** This method on `ScatterSplitReplacementModel` implements ops in model after. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `ScatterSplitReplacementModel` 中的这个方法实现了 ops in model after。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_scatter_split_replace` (lines 65-110)
```python
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
def test_scatter_split_replace(dtype):
    torch.set_default_device(DEVICE_TYPE)
    torch.set_default_dtype(dtype)
    torch.manual_seed(0)

    num_heads = 8
    num_kv_heads = 4
    head_size = 64

    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=["+rotary_embedding"],
        ),
    )
    with vllm.config.set_current_vllm_config(vllm_config):
        # ScatterSplitReplacementPass requires SplitCoalescingPass to be run before it
        coalesce_pass = SplitCoalescingPass(vllm_config)
        replace_pass = ScatterSplitReplacementPass(vllm_config)
        passes = [coalesce_pass, replace_pass]
        backend = TestBackend(*passes)

        model = ScatterSplitReplacementModel(num_heads, num_kv_heads, head_size, dtype)

        T = 5
        qkv = torch.randn(
            T, num_heads * head_size + 2 * num_kv_heads * head_size, dtype=dtype
        )
        pos = torch.arange(T, dtype=torch.long)

        qkv_eager = qkv.clone()
        pos_eager = pos.clone()
        result_eager = model(qkv_eager, pos_eager)

        torch._dynamo.mark_dynamic(qkv, 0)
        torch._dynamo.mark_dynamic(pos, 0)

        model_compiled = torch.compile(model, backend=backend)
        result_compiled = model_compiled(qkv, pos)

        for eager, compiled in zip(result_eager, result_compiled):
            torch.testing.assert_close(eager, compiled)

        assert backend.op_count(torch.ops.aten.slice_scatter.default) == 0
        assert backend.op_count(torch.ops.aten.split_with_sizes.default) == 1
```
**EN:** This pytest case verifies scatter split replace. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as dtype. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 scatter split replace 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 dtype 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch.nn`
- `vllm`
- `tests.compile.backend -> TestBackend`
- `vllm.compilation.passes.utility.scatter_split_replace -> ScatterSplitReplacementPass`
- `vllm.compilation.passes.utility.split_coalescing -> SplitCoalescingPass`
- `vllm.config -> CompilationConfig, CompilationMode, VllmConfig`
- `vllm.model_executor.layers.rotary_embedding -> RotaryEmbedding`
- `vllm.platforms -> current_platform`
