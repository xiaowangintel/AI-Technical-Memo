# test_simple.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fullgraph/test_simple.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / fullgraph / test_simple, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / fullgraph / test_simple 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""
Test the piecewise compilation with a simple model so that we
can exactly calculate the expected output and side effects.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-27)
```python
import pytest
import torch
from torch import nn

from vllm.compilation.counter import compilation_counter
from vllm.compilation.decorators import support_torch_compile
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    CUDAGraphMode,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.forward_context import BatchDescriptor, set_forward_context
from vllm.utils.torch_utils import is_torch_equal_or_newer

from ...utils import create_new_process_for_each_test

# This import automatically registers `torch.ops.silly.attention`
from ..silly_attention import get_global_counter, reset_global_counter
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from ...utils, ..silly_attention; and vLLM components like vllm.compilation.counter, vllm.compilation.decorators, vllm.config, vllm.forward_context.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 ...utils、..silly_attention；vLLM 内部组件，例如 vllm.compilation.counter、vllm.compilation.decorators、vllm.config、vllm.forward_context。

### Function `foo` (lines 31-33)
```python
@torch.library.custom_op("mylib::foo", mutates_args=())
def foo(x: torch.Tensor) -> int:
    return 3
```
**EN:** This helper function implements the shared logic for foo. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 foo 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_` (lines 36-38)
```python
@foo.register_fake
def _(x):
    return torch.library.get_ctx().new_dynamic_size()
```
**EN:** This helper function implements the shared logic for . It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了  所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `SillyModel` (lines 41-42)
```python
@support_torch_compile
class SillyModel(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for SillyModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 SillyModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `SillyModel.__init__` (lines 43-52)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        intermediate_unbacked=False,
        **kwargs,
    ) -> None:
        super().__init__()
        self.intermediate_unbacked = intermediate_unbacked
```
**EN:** This method implements the initialization for `SillyModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `SillyModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `SillyModel.forward` (lines 54-78)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """
        Overall effect:
        x = 3 * x + 19
        global_counter += 2
        """
        x = x + 1
        x = x + 2
        out = torch.empty_like(x)
        torch.ops.silly.attention(x, x, x, out)
        x = out
        x = x - 2

        if self.intermediate_unbacked:
            # Test for unbacked symints: the following is a fancy way to multiply by 1
            u0 = foo(x)
            ones = x.new_ones(x.shape[0], u0).sum(-1) / 3
            x = x * ones

        x = x - 1
        out = torch.empty_like(x)
        torch.ops.silly.attention(x, x, x, out)
        x = out
        x = x + 1
        return x
```
**EN:** This method on `SillyModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `SillyModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_run_simple_model` (lines 81-157)
```python
@torch._dynamo.config.patch(capture_dynamic_output_shape_ops=True)
def _run_simple_model(
    splitting_ops,
    use_inductor_graph_partition,
    backend,
    expected_num_piecewise_graphs_seen,
    expected_num_piecewise_capturable_graphs_seen,
    expected_num_backend_compilations,
    expected_num_cudagraph_captured,
    *,
    intermediate_unbacked=False,
):
    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            backend=backend,
            splitting_ops=splitting_ops,
            use_inductor_graph_partition=use_inductor_graph_partition,
            cudagraph_copy_inputs=True,
            cudagraph_capture_sizes=[1, 2],
        )
    )
    with set_current_vllm_config(vllm_config):
        model = SillyModel(
            vllm_config=vllm_config,
            prefix="",
            intermediate_unbacked=intermediate_unbacked,
        )

    inputs = torch.randn(100).cuda()

    with (
        compilation_counter.expect(
            num_graphs_seen=1,  # one graph for the model
            num_piecewise_graphs_seen=expected_num_piecewise_graphs_seen,
            num_piecewise_capturable_graphs_seen=expected_num_piecewise_capturable_graphs_seen,
            num_backend_compilations=expected_num_backend_compilations,
            num_cudagraph_captured=expected_num_cudagraph_captured,
        ),
        set_forward_context(None, vllm_config=vllm_config),
    ):  # background context
        # warm up with background context
        model(inputs)

        # capturing/replaying should under context of cudagraph dispatching
        with set_forward_context(
            None,
            vllm_config=vllm_config,
            cudagraph_runtime_mode=CUDAGraphMode.PIECEWISE,
            batch_descriptor=BatchDescriptor(
                num_tokens=2,
            ),
        ):
            model(torch.randn(2).cuda())
        with set_forward_context(
            None,
            vllm_config=vllm_config,
            cudagraph_runtime_mode=CUDAGraphMode.PIECEWISE,
            batch_descriptor=BatchDescriptor(
                num_tokens=1,
            ),
        ):
            model(torch.randn(1).cuda())

        input = torch.zeros(2).cuda()
        reset_global_counter()
        with set_forward_context(
            None,
            vllm_config=vllm_config,
            cudagraph_runtime_mode=CUDAGraphMode.PIECEWISE,
            batch_descriptor=BatchDescriptor(
                num_tokens=2,
            ),
        ):
            output = model(input)
        assert get_global_counter() == 2
        assert torch.allclose(output.cpu(), torch.tensor([19.0, 19.0]))
```
**EN:** This helper function implements the shared logic for run simple model. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 run simple model 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_simple_piecewise_compile` (lines 160-185)
```python
@pytest.mark.parametrize("backend", ["inductor", "eager"])
@pytest.mark.parametrize("intermediate_unbacked", [True, False])
@torch.inference_mode()
@create_new_process_for_each_test("spawn")
def test_simple_piecewise_compile(backend, intermediate_unbacked, monkeypatch):
    # `intermediate_unbacked` flips a control-flow branch inside
    # `SillyModel.forward`, but the AOT-compile cache key only hashes the
    # forward function's qualname + line number, so both parametrize variants
    # share the same cache slot. Disabling the cache forces each variant to
    # compile fresh; otherwise the second-running variant loads the first's
    # artifact and segfaults with an illegal memory access.
    monkeypatch.setenv("VLLM_DISABLE_COMPILE_CACHE", "1")
    _run_simple_model(
        splitting_ops=["silly::attention"],
        use_inductor_graph_partition=False,
        backend=backend,
        # 2 * num_layers + 1
        expected_num_piecewise_graphs_seen=5,
        # 1 + num_layers
        expected_num_piecewise_capturable_graphs_seen=3,
        # num_piecewise_capturable_graphs_seen
        expected_num_backend_compilations=3,
        # num_cudagraph_sizes * num_piecewise_capturable_graphs_seen
        expected_num_cudagraph_captured=6,
        intermediate_unbacked=intermediate_unbacked,
    )
```
**EN:** This pytest case verifies simple piecewise compile. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as backend, intermediate_unbacked, monkeypatch.
**CN:** 该 pytest 用例验证 simple piecewise compile 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 backend、intermediate_unbacked、monkeypatch 等 fixture 或输入。

### Function `test_simple_inductor_graph_partition` (lines 188-209)
```python
@torch.inference_mode()
def test_simple_inductor_graph_partition(monkeypatch):
    if not is_torch_equal_or_newer("2.9.0.dev"):
        pytest.skip("inductor graph partition is only available in PyTorch 2.9+")

    # disable compile cache so that we run separately for different splitting_ops
    # and get the expected number of cudagraphs captured.
    monkeypatch.setenv("VLLM_DISABLE_COMPILE_CACHE", "1")

    _run_simple_model(
        splitting_ops=["silly::attention"],
        use_inductor_graph_partition=True,
        backend="inductor",
        # Since not splitting at fx graph level
        expected_num_piecewise_graphs_seen=1,
        # Since not splitting at fx graph level
        expected_num_piecewise_capturable_graphs_seen=1,
        # Since not splitting at fx graph level
        expected_num_backend_compilations=1,
        # Inductor graph partition still captures 6 graph, same as fx graph partition
        expected_num_cudagraph_captured=6,
    )
```
**EN:** This pytest case verifies simple inductor graph partition. it consumes fixtures or inputs such as monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 simple inductor graph partition 的行为。 它会使用诸如 monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch -> nn`
- `vllm.compilation.counter -> compilation_counter`
- `vllm.compilation.decorators -> support_torch_compile`
- `vllm.config -> CompilationConfig, CompilationMode, CUDAGraphMode, VllmConfig, set_current_vllm_config`
- `vllm.forward_context -> BatchDescriptor, set_forward_context`
- `vllm.utils.torch_utils -> is_torch_equal_or_newer`
- `...utils -> create_new_process_for_each_test`
- `..silly_attention -> get_global_counter, reset_global_counter`
