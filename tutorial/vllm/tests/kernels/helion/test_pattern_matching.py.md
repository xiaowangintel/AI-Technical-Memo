# test_pattern_matching.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/helion/test_pattern_matching.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / helion / test_pattern_matching, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / helion / test_pattern_matching 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Test make_fx tracing and inductor pattern matching with HelionKernelWrapper."""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 5-11)
```python
import contextlib
from unittest.mock import Mock, patch

import pytest
import torch

from vllm.utils.import_utils import has_helion
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as contextlib, unittest.mock, pytest, torch; and vLLM components like vllm.utils.import_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 contextlib、unittest.mock、pytest、torch；vLLM 内部组件，例如 vllm.utils.import_utils。

### Top-level block starting at line 13 (lines 13-17)
```python
if not has_helion():
    pytest.skip(
        "Helion is not installed. Install with: pip install vllm[helion]",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 19-21)
```python
import helion
import helion.language as hl
from helion._compat import requires_torch_version
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as helion, helion.language, helion._compat.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 helion、helion.language、helion._compat。

### Top-level block starting at line 23 (lines 23-27)
```python
if not requires_torch_version("2.11"):
    pytest.skip(
        "HigherOrderOp requires PyTorch >= 2.11",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 29-42)
```python
from helion._compiler._dynamo.higher_order_ops import (
    helion_kernel_side_table,
    helion_kernel_wrapper_mutation,
)
from torch._inductor.pattern_matcher import (
    PatternMatcherPass,
    fwd_only,
    register_replacement,
    select_decomp_table,
)
from torch.fx.experimental.proxy_tensor import make_fx

from vllm.kernels.helion.config_manager import ConfigManager
from vllm.kernels.helion.register import HelionKernelWrapper
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as helion._compiler._dynamo.higher_order_ops, torch._inductor.pattern_matcher, torch.fx.experimental.proxy_tensor; and vLLM components like vllm.kernels.helion.config_manager, vllm.kernels.helion.register.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 helion._compiler._dynamo.higher_order_ops、torch._inductor.pattern_matcher、torch.fx.experimental.proxy_tensor；vLLM 内部组件，例如 vllm.kernels.helion.config_manager、vllm.kernels.helion.register。

### Function `_helion_mock_context` (lines 45-63)
```python
@contextlib.contextmanager
def _helion_mock_context():
    configs = {
        "default": helion.Config(block_sizes=[64], num_warps=2, num_stages=2),
    }
    mock_config_manager = Mock(spec=ConfigManager)
    mock_config_manager.get_platform_configs = Mock(return_value=configs)

    with (
        patch(
            "vllm.kernels.helion.config_manager.ConfigManager",
            return_value=mock_config_manager,
        ),
        patch(
            "vllm.kernels.helion.utils.get_canonical_gpu_name",
            return_value="nvidia_h200",
        ),
    ):
        yield
```
**EN:** This helper function implements the shared logic for helion mock context. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 helion mock context 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestMakeFxHop` (lines 66-66)
```python
class TestMakeFxHop:
```
**EN:** This helper class groups the state and behavior needed for TestMakeFxHop. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestMakeFxHop 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestMakeFxHop.setup_method` (lines 67-68)
```python
    def setup_method(self):
        helion_kernel_side_table.reset_table()
```
**EN:** This method on `TestMakeFxHop` implements setup method. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestMakeFxHop` 中的这个方法实现了 setup method。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestMakeFxHop.test_make_fx_symbolic` (lines 70-130)
```python
    @pytest.mark.skip(reason="SymInt proxy tracking issue with PyTorch 2.11+")
    def test_make_fx_symbolic(self):
        def raw_add_scale(
            x: torch.Tensor, y: torch.Tensor, scale: float
        ) -> tuple[torch.Tensor, int, torch.Tensor]:
            out_x = torch.empty_like(x)
            out_y = torch.empty_like(x)
            for tile in hl.tile(x.size()):
                out_x[tile] = x[tile] + y[tile] * scale
                out_y[tile] = out_x[tile] * 2.0
            return out_x, 42, out_y

        input_x = torch.randn(7, 13)
        input_y = torch.randn(7, 13)
        scale = 0.5

        with _helion_mock_context():
            wrapper = HelionKernelWrapper(
                raw_kernel_func=raw_add_scale,
                op_name="test_make_fx",
                fake_impl=lambda *a, **kw: None,
                config_picker=lambda args, keys: "default",
            )

            def fn(x, y):
                return wrapper(x, y, scale)

            gm = make_fx(fn, tracing_mode="symbolic")(input_x, input_y)

        hop_nodes = [
            n
            for n in gm.graph.nodes
            if n.op == "call_function" and n.target is helion_kernel_wrapper_mutation
        ]
        assert len(hop_nodes) == 1
        node = hop_nodes[0]

        assert node.kwargs["constant_args"]["scale"] == scale
        assert set(node.kwargs["tensor_args"]) == {"x", "y"}

        specs = node.kwargs["output_spec"]["leaf_specs"]
        tensor_specs = [s for s in specs if s["type"] == "tensor"]
        scalar_specs = [s for s in specs if s["type"] == "scalar"]
        assert len(tensor_specs) == 2
        assert len(scalar_specs) == 1

        for spec in tensor_specs:
            assert spec["dtype"] == input_x.dtype

        assert scalar_specs[0]["scalar_value"] == 42

        for val in node.meta["val"]:
            assert all(isinstance(s, torch.SymInt) for s in val.shape)

        # Both out_x and out_y are empty_like(x), so output shapes == input shape
        input_node = next(n for n in gm.graph.nodes if n.op == "placeholder")
        input_shape = input_node.meta["val"].shape
        for val in node.meta["val"]:
            assert len(val.shape) == len(input_shape)
            for out_s, in_s in zip(val.shape, input_shape):
                assert out_s == in_s
```
**EN:** This method on `TestMakeFxHop` checks make fx symbolic. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestMakeFxHop` 中的这个方法用于检查 make fx symbolic。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestMakeFxHop.test_pattern_matcher_replaces_with_helion_hop` (lines 132-205)
```python
    @pytest.mark.skip(reason="SymInt proxy tracking issue with PyTorch 2.11+")
    def test_pattern_matcher_replaces_with_helion_hop(self):
        def raw_silu_mul(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
            M, N = x.size()
            out = torch.empty_like(x)
            for tile_m, tile_n in hl.tile([M, N]):
                out[tile_m, tile_n] = (
                    torch.nn.functional.silu(x[tile_m, tile_n]) * y[tile_m, tile_n]
                )
            return out

        with _helion_mock_context():
            wrapper = HelionKernelWrapper(
                raw_kernel_func=raw_silu_mul,
                op_name="test_pm_silu_mul",
                fake_impl=lambda *a, **kw: None,
                config_picker=lambda args, keys: "default",
            )

            def pattern(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
                return torch.nn.functional.silu(x) * y

            def replacement(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
                return wrapper(x, y)

            inputs = [torch.randn(8, 16), torch.randn(8, 16)]

            pm_pass = PatternMatcherPass(pass_name="test_helion_replacement")
            register_replacement(pattern, replacement, inputs, fwd_only, pm_pass)

            def model(x, y):
                return torch.nn.functional.silu(x) * y

            decompositions = select_decomp_table()
            input_x = torch.randn(8, 16)
            input_y = torch.randn(8, 16)
            gm = make_fx(model, decompositions, tracing_mode="symbolic")(
                input_x, input_y
            )

            def count_hop_nodes(graph):
                return sum(
                    1
                    for n in graph.nodes
                    if n.op == "call_function"
                    and n.target is helion_kernel_wrapper_mutation
                )

            assert count_hop_nodes(gm.graph) == 0

            match_count = pm_pass.apply(gm.graph)
            gm.graph.lint()
            gm.recompile()

            assert match_count == 1
            assert count_hop_nodes(gm.graph) == 1

            hop_node = next(
                n
                for n in gm.graph.nodes
                if n.op == "call_function"
                and n.target is helion_kernel_wrapper_mutation
            )

            # raw_silu_mul returns empty_like(x), so output shape == input shape
            for val in hop_node.meta["val"]:
                assert all(isinstance(s, torch.SymInt) for s in val.shape)

            input_node = next(n for n in gm.graph.nodes if n.op == "placeholder")
            input_shape = input_node.meta["val"].shape
            output_shape = hop_node.meta["val"][0].shape
            assert len(output_shape) == len(input_shape)
            for out_s, in_s in zip(output_shape, input_shape):
                assert out_s == in_s
```
**EN:** This method on `TestMakeFxHop` checks pattern matcher replaces with helion hop. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestMakeFxHop` 中的这个方法用于检查 pattern matcher replaces with helion hop。 结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `contextlib`
- `unittest.mock -> Mock, patch`
- `pytest`
- `torch`
- `vllm.utils.import_utils -> has_helion`
- `helion`
- `helion.language`
- `helion._compat -> requires_torch_version`
- `helion._compiler._dynamo.higher_order_ops -> helion_kernel_side_table, helion_kernel_wrapper_mutation`
- `torch._inductor.pattern_matcher -> PatternMatcherPass, fwd_only, register_replacement, select_decomp_table`
- `torch.fx.experimental.proxy_tensor -> make_fx`
- `vllm.kernels.helion.config_manager -> ConfigManager`
- `vllm.kernels.helion.register -> HelionKernelWrapper`
