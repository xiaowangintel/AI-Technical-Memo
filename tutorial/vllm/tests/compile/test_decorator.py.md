# test_decorator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/test_decorator.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / test_decorator, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / test_decorator 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-21)
```python
import pytest
import torch
from torch import nn

from vllm.compilation.counter import compilation_counter
from vllm.compilation.decorators import ignore_torch_compile, support_torch_compile
from vllm.config import (
    CacheConfig,
    CompilationConfig,
    CompilationMode,
    CUDAGraphMode,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.forward_context import BatchDescriptor, set_forward_context
from vllm.utils.torch_utils import is_torch_equal_or_newer

# This import automatically registers `torch.ops.silly.attention`
from . import silly_attention  # noqa: F401
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from .; and vLLM components like vllm.compilation.counter, vllm.compilation.decorators, vllm.config, vllm.forward_context.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 .；vLLM 内部组件，例如 vllm.compilation.counter、vllm.compilation.decorators、vllm.config、vllm.forward_context。

### Constants and module state (lines 23-24)
```python
BATCH_SIZE = 32
MLP_SIZE = 128
```
**EN:** This block centralizes shared constants and parameter grids, including BATCH_SIZE, MLP_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 BATCH_SIZE、MLP_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `run_model` (lines 27-67)
```python
@torch.inference_mode
def run_model(
    vllm_config: VllmConfig, model: nn.Module, cudagraph_runtime_mode: CUDAGraphMode
):
    with set_forward_context({}, vllm_config=vllm_config):
        # warmup for the model with cudagraph_mode NONE
        model(torch.randn(BATCH_SIZE, MLP_SIZE).cuda())

        # simulate cudagraphs capturing
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=2,
            ),
        ):
            model(torch.randn(2, MLP_SIZE).cuda())
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=1,
            ),
        ):
            model(torch.randn(1, MLP_SIZE).cuda())

        # simulate cudagraphs replay
        with set_forward_context(
            {},
            vllm_config=vllm_config,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            batch_descriptor=BatchDescriptor(
                num_tokens=2,
            ),
        ):
            output = model(torch.randn(2, MLP_SIZE).cuda())

        output = output.cpu()
        return output.cpu()
```
**EN:** This helper function implements the shared logic for run model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 run model 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_ignore_torch_compile_decorator` (lines 70-161)
```python
@pytest.mark.parametrize("use_inductor_graph_partition", [True, False])
def test_ignore_torch_compile_decorator(use_inductor_graph_partition, monkeypatch):
    # disable compile cache so that we can count the number of compilations
    # appropriately
    monkeypatch.setenv("VLLM_DISABLE_COMPILE_CACHE", "1")

    if use_inductor_graph_partition and not is_torch_equal_or_newer("2.9.0.dev"):
        pytest.skip("inductor graph partition is only available in PyTorch 2.9+")

    # piecewise
    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            splitting_ops=["silly::attention"],
            cudagraph_capture_sizes=[1, 2],
            use_inductor_graph_partition=use_inductor_graph_partition,
        )
    )
    cudagraph_runtime_mode = CUDAGraphMode.PIECEWISE

    expected_num_graphs_seen = 1
    expected_num_cudagraph_captured = (
        4  # num_cudagraph_sizes * num cudagraphs to capture
    )
    if use_inductor_graph_partition:
        expected_num_piecewise_graphs_seen = 1
        expected_num_piecewise_capturable_graphs_seen = 1
        expected_num_backend_compilations = 1
    else:
        expected_num_piecewise_graphs_seen = 3
        expected_num_piecewise_capturable_graphs_seen = 2
        expected_num_backend_compilations = 2

    @support_torch_compile
    class A(nn.Module):
        def __init__(
            self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs
        ) -> None:
            super().__init__()

# ... excerpt ...

    with set_current_vllm_config(vllm_config):
        mod_A = A(vllm_config=vllm_config, prefix="").eval().cuda()

    # A has support_torch_compile
    with compilation_counter.expect(
        num_graphs_seen=expected_num_graphs_seen,
        num_piecewise_graphs_seen=expected_num_piecewise_graphs_seen,
        num_piecewise_capturable_graphs_seen=expected_num_piecewise_capturable_graphs_seen,
        num_backend_compilations=expected_num_backend_compilations,
        num_cudagraph_captured=expected_num_cudagraph_captured,
    ):
        run_model(vllm_config, mod_A, cudagraph_runtime_mode)

    with set_current_vllm_config(vllm_config):
        mod_B = B(vllm_config=vllm_config, prefix="").eval().cuda()

    # B's ignore_torch_compile should override A's support_torch_compile
    with compilation_counter.expect(
        num_graphs_seen=0,
        num_piecewise_graphs_seen=0,
        num_piecewise_capturable_graphs_seen=0,
        num_backend_compilations=0,
        num_cudagraph_captured=0,
    ):
        run_model(vllm_config, mod_B, cudagraph_runtime_mode)

    with set_current_vllm_config(vllm_config):
        mod_C = C(vllm_config=vllm_config, prefix="").eval().cuda()

    # C's support_torch_compile should override B's ignore_torch_compile
    with compilation_counter.expect(
        num_graphs_seen=expected_num_graphs_seen,
        num_piecewise_graphs_seen=expected_num_piecewise_graphs_seen,
        num_piecewise_capturable_graphs_seen=expected_num_piecewise_capturable_graphs_seen,
        num_backend_compilations=expected_num_backend_compilations,
        num_cudagraph_captured=expected_num_cudagraph_captured,
    ):
        run_model(vllm_config, mod_C, cudagraph_runtime_mode)
```
**EN:** This pytest case verifies ignore torch compile decorator. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as use_inductor_graph_partition, monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 ignore torch compile decorator 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 use_inductor_graph_partition、monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。 由于该代码块较大，这里只展示关键片段。

### Class `B` (lines 166-169)
```python
@support_torch_compile(
    enable_if=lambda vllm_config: vllm_config.cache_config.kv_sharing_fast_prefill
)
class B(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for B. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 B 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `B.__init__` (lines 170-171)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs) -> None:
        super().__init__()
```
**EN:** This method implements the initialization for `B`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `B` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `B.forward` (lines 173-179)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = x + x
        attn_output = torch.empty_like(x)
        torch.ops.silly.attention(x, x, x, attn_output)
        x = attn_output
        x = x + x
        return x
```
**EN:** This method on `B` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `B` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `A` (lines 184-187)
```python
@support_torch_compile(
    enable_if=lambda vllm_config: not vllm_config.cache_config.kv_sharing_fast_prefill
)
class A(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for A. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 A 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `A.__init__` (lines 188-191)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs) -> None:
        super().__init__()
        self.mod1 = B(vllm_config=vllm_config, prefix=prefix, **kwargs)
        self.mod2 = B(vllm_config=vllm_config, prefix=prefix, **kwargs)
```
**EN:** This method implements the initialization for `A`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `A` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `A.forward` (lines 193-199)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = self.mod1(x)
        attn_output = torch.empty_like(x)
        torch.ops.silly.attention(x, x, x, attn_output)
        x = attn_output
        x = self.mod2(x)
        return x
```
**EN:** This method on `A` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `A` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_conditional_compile_enable_if` (lines 202-286)
```python
@pytest.mark.parametrize("use_inductor_graph_partition", [True, False])
def test_conditional_compile_enable_if(use_inductor_graph_partition, monkeypatch):
    # disable compile cache so that we can count the number of compilations
    # appropriately
    monkeypatch.setenv("VLLM_DISABLE_COMPILE_CACHE", "1")

    if use_inductor_graph_partition and not is_torch_equal_or_newer("2.9.0.dev"):
        pytest.skip("inductor graph partition is only available in PyTorch 2.9+")

    vllm_config = VllmConfig(
        cache_config=CacheConfig(
            kv_sharing_fast_prefill=True,
        ),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            splitting_ops=["silly::attention"],
            cudagraph_capture_sizes=[1, 2],
            use_inductor_graph_partition=use_inductor_graph_partition,
        ),
    )
    cudagraph_runtime_mode = CUDAGraphMode.PIECEWISE

    with set_current_vllm_config(vllm_config):
        mod_A = A(vllm_config=vllm_config, prefix="").eval().cuda()

    if use_inductor_graph_partition:
        expected_num_piecewise_graphs_seen = 2
        expected_num_piecewise_capturable_graphs_seen = 2
        expected_num_backend_compilations = 2
    else:
        expected_num_piecewise_graphs_seen = 6
        expected_num_piecewise_capturable_graphs_seen = 4
        expected_num_backend_compilations = 4

    # A has support_torch_compile but enable_if fn returns False
    # enable_if will be True for B, so we expect mod1 and mod2
    # to be compiled
    with compilation_counter.expect(
        num_graphs_seen=2,
        num_piecewise_graphs_seen=expected_num_piecewise_graphs_seen,
# ... excerpt ...
        run_model(vllm_config, mod_A, cudagraph_runtime_mode)

    # Set kv_sharing_fast_prefill=False
    # which will cause A to be compiled and B to not be compiled
    vllm_config = VllmConfig(
        cache_config=CacheConfig(
            kv_sharing_fast_prefill=False,
        ),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            splitting_ops=["silly::attention"],
            cudagraph_capture_sizes=[1, 2],
            use_inductor_graph_partition=use_inductor_graph_partition,
        ),
    )

    with set_current_vllm_config(vllm_config):
        mod_A = A(vllm_config=vllm_config, prefix="").eval().cuda()

    if use_inductor_graph_partition:
        expected_num_piecewise_graphs_seen = 1
        expected_num_piecewise_capturable_graphs_seen = 1
        expected_num_backend_compilations = 1
    else:
        # 3 attn ops and 4 non-attn ops
        expected_num_piecewise_graphs_seen = 7
        expected_num_piecewise_capturable_graphs_seen = 4
        expected_num_backend_compilations = 4

    with compilation_counter.expect(
        num_graphs_seen=1,
        num_piecewise_graphs_seen=expected_num_piecewise_graphs_seen,
        # 3 attn ops and 4 non-attn ops
        num_piecewise_capturable_graphs_seen=expected_num_piecewise_capturable_graphs_seen,
        num_backend_compilations=expected_num_backend_compilations,
        num_cudagraph_captured=8,
        # num_cudagraph_sizes * num cudagraphable graphs to capture
    ):
        run_model(vllm_config, mod_A, cudagraph_runtime_mode)
```
**EN:** This pytest case verifies conditional compile enable if. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as use_inductor_graph_partition, monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 conditional compile enable if 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 use_inductor_graph_partition、monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch -> nn`
- `vllm.compilation.counter -> compilation_counter`
- `vllm.compilation.decorators -> ignore_torch_compile, support_torch_compile`
- `vllm.config -> CacheConfig, CompilationConfig, CompilationMode, CUDAGraphMode, VllmConfig, set_current_vllm_config`
- `vllm.forward_context -> BatchDescriptor, set_forward_context`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer`
- `. -> silly_attention`
