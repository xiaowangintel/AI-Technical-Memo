# test_compile_ranges.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/test_compile_ranges.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / test_compile_ranges, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / test_compile_ranges 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-24)
```python
from typing import Any

import torch
from torch import fx as fx
from torch import nn

# This import automatically registers `torch.ops.silly.attention`
import tests.compile.silly_attention  # noqa
from vllm.compilation.counter import compilation_counter
from vllm.compilation.decorators import support_torch_compile
from vllm.compilation.passes.inductor_pass import (
    InductorPass,
    get_pass_context,
)
from vllm.config import (
    VllmConfig,
    set_current_vllm_config,
)
from vllm.config.compilation import CompilationConfig, CompilationMode
from vllm.config.scheduler import SchedulerConfig
from vllm.config.utils import Range
from vllm.forward_context import set_forward_context
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as typing, torch; shared test helpers from tests.compile.silly_attention  # noqa; and vLLM components like vllm.compilation.counter, vllm.compilation.decorators, vllm.compilation.passes.inductor_pass, vllm.config.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 typing、torch；共享测试辅助模块，例如 tests.compile.silly_attention  # noqa；vLLM 内部组件，例如 vllm.compilation.counter、vllm.compilation.decorators、vllm.compilation.passes.inductor_pass、vllm.config。

### Constants and module state (lines 26-27)
```python
BATCH_SIZE = 64
MLP_SIZE = 128
```
**EN:** This block centralizes shared constants and parameter grids, including BATCH_SIZE, MLP_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 BATCH_SIZE、MLP_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `TestModel` (lines 30-31)
```python
@support_torch_compile
class TestModel(nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestModel.__init__` (lines 32-33)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs) -> None:
        super().__init__()
```
**EN:** This method implements the initialization for `TestModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestModel.forward` (lines 35-41)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = x + x
        attn_output = torch.empty_like(x)
        torch.ops.silly.attention(x, x, x, attn_output)
        x = attn_output
        x = x * 3
        return x
```
**EN:** This method on `TestModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `run_model` (lines 44-49)
```python
@torch.inference_mode
def run_model(vllm_config: VllmConfig, model: nn.Module, batch_sizes: list[int]):
    with set_forward_context({}, vllm_config=vllm_config):
        model(torch.randn(BATCH_SIZE, MLP_SIZE))
        for batch_size in batch_sizes:
            model(torch.randn(batch_size, MLP_SIZE))
```
**EN:** This helper function implements the shared logic for run model. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 run model 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `PostGradRangeChecker` (lines 52-52)
```python
class PostGradRangeChecker(InductorPass):
```
**EN:** This helper class groups the state and behavior needed for PostGradRangeChecker. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 PostGradRangeChecker 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `PostGradRangeChecker.__init__` (lines 53-55)
```python
    def __init__(self, ranges: list[Range]):
        self.ranges = ranges
        self.num_calls = 0
```
**EN:** This method implements the initialization for `PostGradRangeChecker`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `PostGradRangeChecker` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `PostGradRangeChecker.__call__` (lines 57-62)
```python
    def __call__(self, graph: fx.Graph):
        compile_range = get_pass_context().compile_range
        assert compile_range in self.ranges, (
            f"Compile range {compile_range} not in {self.ranges}"
        )
        self.num_calls += 1
```
**EN:** This method implements the call entry for `PostGradRangeChecker`. assertions at the end lock in the intended behavior or graph shape.
**CN:** 这个方法实现了 `PostGradRangeChecker` 的调用入口。 结尾处的断言会固定预期行为或计算图形态。

### Method `PostGradRangeChecker.uuid` (lines 64-66)
```python
    def uuid(self) -> str:
        state: dict[str, Any] = {}
        return InductorPass.hash_dict(state)
```
**EN:** This method on `PostGradRangeChecker` implements uuid. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `PostGradRangeChecker` 中的这个方法实现了 uuid。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_compile_ranges` (lines 69-108)
```python
def test_compile_ranges(use_fresh_inductor_cache):
    post_grad_range_checker = PostGradRangeChecker(
        [
            Range(start=1, end=8),
            Range(start=16, end=16),
            Range(start=9, end=32),
            Range(start=64, end=64),
            Range(start=128, end=128),
            Range(start=33, end=8192),
        ]
    )
    torch.set_default_device("cuda")
    vllm_config = VllmConfig(
        scheduler_config=SchedulerConfig(
            max_num_batched_tokens=8192,
            max_model_len=8192,
            is_encoder_decoder=False,
        ),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            compile_ranges_endpoints=[8, 32],
            compile_sizes=[16, 64, 128],
            inductor_compile_config={
                "post_grad_custom_post_pass": post_grad_range_checker,
            },
        ),
    )

    with set_current_vllm_config(vllm_config):
        model = TestModel(vllm_config=vllm_config, prefix="").eval()
        # Number of compilations: 3 compile ranges + 3 compile sizes
        batch_sizes = [1, 4, 16, 24, 48, 64, 8192]

        with compilation_counter.expect(
            num_graphs_seen=1,
            num_piecewise_graphs_seen=1,
            num_backend_compilations=6,
        ):
            run_model(vllm_config, model, batch_sizes)
        assert post_grad_range_checker.num_calls == 6
```
**EN:** This pytest case verifies compile ranges. it consumes fixtures or inputs such as use_fresh_inductor_cache. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 compile ranges 的行为。 它会使用诸如 use_fresh_inductor_cache 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_compile_config_get_compile_ranges` (lines 111-127)
```python
def test_compile_config_get_compile_ranges():
    compilation_config = CompilationConfig(
        compile_ranges_endpoints=[8, 32],
    )
    VllmConfig(
        scheduler_config=SchedulerConfig(
            max_num_batched_tokens=8192,
            max_model_len=8192,
            is_encoder_decoder=False,
        ),
        compilation_config=compilation_config,
    )
    assert compilation_config.get_compile_ranges() == [
        Range(start=1, end=8),
        Range(start=9, end=32),
        Range(start=33, end=8192),
    ]
```
**EN:** This pytest case verifies compile config get compile ranges. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 compile config get compile ranges 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Class `PostGradStaticShapeChecker` (lines 130-133)
```python
class PostGradStaticShapeChecker(InductorPass):
    """Asserts that compile_sizes entries produce graphs with fully concrete
    (non-symbolic) shapes, and compile_ranges entries have symbolic shapes."""
```
**EN:** This helper class groups the state and behavior needed for PostGradStaticShapeChecker. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 PostGradStaticShapeChecker 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `PostGradStaticShapeChecker.__init__` (lines 134-136)
```python
    def __init__(self):
        self.num_static_calls = 0
        self.num_dynamic_calls = 0
```
**EN:** This method implements the initialization for `PostGradStaticShapeChecker`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `PostGradStaticShapeChecker` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `PostGradStaticShapeChecker.__call__` (lines 138-164)
```python
    def __call__(self, graph: fx.Graph):
        from torch.fx.experimental.symbolic_shapes import is_symbolic

        compile_range = get_pass_context().compile_range
        is_single = compile_range.is_single_size()

        for node in graph.nodes:
            val = node.meta.get("val")
            if val is None:
                val = node.meta.get("example_value")
            if isinstance(val, torch.Tensor):
                has_symbolic = any(is_symbolic(d) for d in val.shape)
                if is_single:
                    assert not has_symbolic, (
                        f"compile_sizes entry {compile_range}: "
                        f"node '{node.name}' has symbolic shape "
                        f"{val.shape}"
                    )
                else:
                    # compile_ranges should have at least some
                    # symbolic shapes (the batch dimension)
                    if has_symbolic:
                        self.num_dynamic_calls += 1
                        return

        if is_single:
            self.num_static_calls += 1
```
**EN:** This method implements the call entry for `PostGradStaticShapeChecker`. assertions at the end lock in the intended behavior or graph shape.
**CN:** 这个方法实现了 `PostGradStaticShapeChecker` 的调用入口。 结尾处的断言会固定预期行为或计算图形态。

### Method `PostGradStaticShapeChecker.uuid` (lines 166-168)
```python
    def uuid(self) -> str:
        state: dict[str, Any] = {}
        return InductorPass.hash_dict(state)
```
**EN:** This method on `PostGradStaticShapeChecker` implements uuid. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `PostGradStaticShapeChecker` 中的这个方法实现了 uuid。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_compile_sizes_produce_static_shapes` (lines 171-209)
```python
def test_compile_sizes_produce_static_shapes(use_fresh_inductor_cache):
    """Verify that compile_sizes entries are compiled with fully concrete
    shapes (no SymInts), while compile_ranges entries retain dynamic shapes."""
    checker = PostGradStaticShapeChecker()
    torch.set_default_device("cuda")
    vllm_config = VllmConfig(
        scheduler_config=SchedulerConfig(
            max_num_batched_tokens=8192,
            max_model_len=8192,
            is_encoder_decoder=False,
        ),
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            compile_ranges_endpoints=[8],
            compile_sizes=[16],
            inductor_compile_config={
                "post_grad_custom_post_pass": checker,
            },
        ),
    )

    with set_current_vllm_config(vllm_config):
        model = TestModel(vllm_config=vllm_config, prefix="").eval()
        # 3 compilations: Range(1,8), Range(9,8192), single-size 16
        with compilation_counter.expect(
            num_graphs_seen=1,
            num_piecewise_graphs_seen=1,
            num_backend_compilations=3,
        ):
            run_model(vllm_config, model, [1, 16, 64])

    # compile_sizes=16 should produce static shapes
    assert checker.num_static_calls == 1, (
        f"Expected 1 static compilation, got {checker.num_static_calls}"
    )
    # compile_ranges should produce dynamic shapes
    assert checker.num_dynamic_calls == 2, (
        f"Expected 2 dynamic compilations, got {checker.num_dynamic_calls}"
    )
```
**EN:** This pytest case verifies compile sizes produce static shapes. it consumes fixtures or inputs such as use_fresh_inductor_cache. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 compile sizes produce static shapes 的行为。 它会使用诸如 use_fresh_inductor_cache 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_inductor_cache_compile_ranges` (lines 212-257)
```python
def test_inductor_cache_compile_ranges(monkeypatch, use_fresh_inductor_cache):
    # To force multiple compilations, we disable the compile cache
    monkeypatch.setenv("VLLM_DISABLE_COMPILE_CACHE", "1")

    post_grad_range_checker = PostGradRangeChecker(
        ranges=[
            Range(start=1, end=8),
            Range(start=9, end=8192),
        ]
    )
    scheduler_config = SchedulerConfig(
        max_num_batched_tokens=8192,
        max_model_len=8192,
        is_encoder_decoder=False,
    )
    torch.set_default_device("cuda")

    def create_vllm_config():
        return VllmConfig(
            scheduler_config=scheduler_config,
            compilation_config=CompilationConfig(
                mode=CompilationMode.VLLM_COMPILE,
                compile_ranges_endpoints=[8],
                inductor_compile_config={
                    "post_grad_custom_post_pass": post_grad_range_checker,
                },
            ),
        )

    vllm_config_1 = create_vllm_config()
    with set_current_vllm_config(vllm_config_1):
        model1 = TestModel(vllm_config=vllm_config_1, prefix="").eval()
        batch_sizes = [1, 16]
        run_model(vllm_config_1, model1, batch_sizes)
        assert post_grad_range_checker.num_calls == 2

    post_grad_range_checker.num_calls = 0
    # Create a new vllm config with the new pass context
    vllm_config_2 = create_vllm_config()
    with set_current_vllm_config(vllm_config_2):
        model2 = TestModel(vllm_config=vllm_config_2, prefix="").eval()
        batch_sizes = [4, 32]
        run_model(vllm_config_2, model2, batch_sizes)
        # Check that cache is used, so the number of calls
        # should be 0
        assert post_grad_range_checker.num_calls == 0
```
**EN:** This pytest case verifies inductor cache compile ranges. it consumes fixtures or inputs such as monkeypatch, use_fresh_inductor_cache. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 inductor cache compile ranges 的行为。 它会使用诸如 monkeypatch、use_fresh_inductor_cache 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `typing -> Any`
- `torch`
- `torch -> fx`
- `torch -> nn`
- `tests.compile.silly_attention`
- `vllm.compilation.counter -> compilation_counter`
- `vllm.compilation.decorators -> support_torch_compile`
- `vllm.compilation.passes.inductor_pass -> InductorPass, get_pass_context`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.config.compilation -> CompilationConfig, CompilationMode`
- `vllm.config.scheduler -> SchedulerConfig`
- `vllm.config.utils -> Range`
- `vllm.forward_context -> set_forward_context`
