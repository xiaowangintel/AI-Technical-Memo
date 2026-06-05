# test_structured_logging.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/test_structured_logging.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / test_structured_logging, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / test_structured_logging 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-20)
```python
from unittest.mock import patch

import pytest
import regex as re
import torch
from torch import nn

import tests.compile.silly_attention  # noqa
from vllm.compilation.decorators import support_torch_compile
from vllm.config import VllmConfig, set_current_vllm_config
from vllm.config.compilation import (
    CompilationConfig,
    CompilationMode,
    CUDAGraphMode,
)
from vllm.config.scheduler import SchedulerConfig
from vllm.forward_context import set_forward_context
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as unittest.mock, pytest, regex, torch; shared test helpers from tests.compile.silly_attention  # noqa; and vLLM components like vllm.compilation.decorators, vllm.config, vllm.config.compilation, vllm.config.scheduler.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 unittest.mock、pytest、regex、torch；共享测试辅助模块，例如 tests.compile.silly_attention  # noqa；vLLM 内部组件，例如 vllm.compilation.decorators、vllm.config、vllm.config.compilation、vllm.config.scheduler。

### Constants and module state (lines 22-23)
```python
MLP_SIZE = 64
DEVICE_TYPE = current_platform.device_type
```
**EN:** This block centralizes shared constants and parameter grids, including MLP_SIZE, DEVICE_TYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MLP_SIZE、DEVICE_TYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Class `SimpleModel` (lines 26-29)
```python
@support_torch_compile
class SimpleModel(nn.Module):
    """A simple model with a splitting op for piecewise compilation."""
```
**EN:** This helper class groups the state and behavior needed for SimpleModel. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 SimpleModel 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `SimpleModel.__init__` (lines 30-31)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "", **kwargs):
        super().__init__()
```
**EN:** This method implements the initialization for `SimpleModel`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `SimpleModel` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `SimpleModel.forward` (lines 33-38)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = x + x
        attn_output = torch.empty_like(x)
        torch.ops.silly.attention(x, x, x, attn_output)
        x = attn_output * 2
        return x
```
**EN:** This method on `SimpleModel` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `SimpleModel` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TraceStructuredCapture` (lines 41-43)
```python
class TraceStructuredCapture:
    """Captures trace_structured calls for testing."""
```
**EN:** This helper class groups the state and behavior needed for TraceStructuredCapture. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TraceStructuredCapture 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TraceStructuredCapture.__init__` (lines 44-45)
```python
    def __init__(self):
        self.calls: list[dict] = []
```
**EN:** This method implements the initialization for `TraceStructuredCapture`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TraceStructuredCapture` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TraceStructuredCapture.__call__` (lines 47-55)
```python
    def __call__(self, event_type: str, metadata_fn=None, payload_fn=None, **kwargs):
        """Capture a trace_structured call."""
        metadata = metadata_fn() if metadata_fn else {}
        self.calls.append(
            {
                "event_type": event_type,
                "metadata": metadata,
            }
        )
```
**EN:** This method implements the call entry for `TraceStructuredCapture`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TraceStructuredCapture` 的调用入口。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TraceStructuredCapture.get` (lines 57-70)
```python
    def get(self, event_type: str, name_pattern: str) -> list[dict]:
        """Get all calls with the given event type and name matching pattern.

        Args:
            event_type: The event type to filter by (e.g., "artifact", "graph_dump")
            name_pattern: Regex pattern to match against the artifact name
        """
        regex = re.compile(name_pattern)
        return [
            c
            for c in self.calls
            if c["event_type"] == event_type
            and regex.fullmatch(c.get("metadata", {}).get("name", ""))
        ]
```
**EN:** This method on `TraceStructuredCapture` implements get. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TraceStructuredCapture` 中的这个方法实现了 get。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_vllm_structured_logging_artifacts` (lines 73-123)
```python
@pytest.mark.skipif(not torch.cuda.is_available(), reason="CUDA required")
def test_vllm_structured_logging_artifacts(use_fresh_inductor_cache):
    """Test that all expected vLLM artifacts are logged during compilation."""
    torch.set_default_device(DEVICE_TYPE)

    capture = TraceStructuredCapture()

    vllm_config = VllmConfig(
        compilation_config=CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            cudagraph_mode=CUDAGraphMode.PIECEWISE,
            compile_sizes=[8],
            splitting_ops=["silly::attention"],
        ),
        scheduler_config=SchedulerConfig(
            max_num_seqs=8,
            max_model_len=8192,
            is_encoder_decoder=False,
        ),
    )

    # Patch trace_structured to capture calls
    with (
        patch("vllm.compilation.backends.trace_structured", capture),
        patch("vllm.compilation.piecewise_backend.trace_structured", capture),
        set_current_vllm_config(vllm_config),
    ):
        model = SimpleModel(vllm_config=vllm_config, prefix="test")
        with set_forward_context({}, vllm_config=vllm_config):
            model(torch.randn(8, MLP_SIZE))

    config_artifacts = capture.get("artifact", "vllm_compilation_config")
    assert len(config_artifacts) == 1, (
        f"Expected 1 vllm_compilation_config, got {len(config_artifacts)}"
    )
    vllm_piecewise_split_graph = capture.get("graph_dump", "vllm_piecewise_split_graph")
    assert len(vllm_piecewise_split_graph) == 1, (
        "Expected 1 toplevel piecewise split graph, "
        f"got {len(vllm_piecewise_split_graph)}"
    )
    compile_start_artifacts = capture.get("artifact", "vllm_piecewise_compile_start")
    assert len(compile_start_artifacts) == 4, (
        "Expected 4 vllm_piecewise_compile_start "
        "(2 subgraphs x 2 ranges each: dynamic + compile size), "
        f"got {len(compile_start_artifacts)}"
    )
    submod_dumps = capture.get("graph_dump", r"vllm_submod_.*")
    assert len(submod_dumps) == 2, (
        "Expected 2 submods (one before attention, one after attention), "
        f"got {len(submod_dumps)}"
    )
```
**EN:** This pytest case verifies vllm structured logging artifacts. it consumes fixtures or inputs such as use_fresh_inductor_cache. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 vllm structured logging artifacts 的行为。 它会使用诸如 use_fresh_inductor_cache 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `unittest.mock -> patch`
- `pytest`
- `regex`
- `torch`
- `torch -> nn`
- `tests.compile.silly_attention`
- `vllm.compilation.decorators -> support_torch_compile`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.config.compilation -> CompilationConfig, CompilationMode, CUDAGraphMode`
- `vllm.config.scheduler -> SchedulerConfig`
- `vllm.forward_context -> set_forward_context`
- `vllm.platforms -> current_platform`
