# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/layers/mamba/conftest.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This pytest configuration module provides shared fixtures, hooks, or helpers for the layers / mamba tests. / 该 pytest 配置模块为 层 / mamba 测试提供共享夹具、钩子或辅助工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import pytest

from sglang.srt.layers.attention.mamba.ops import ssu_dispatch
from sglang.srt.layers.attention.mamba.ops.ssu_dispatch import (
    initialize_mamba_selective_state_update_backend,
)
from sglang.srt.server_args import ServerArgs
```
**EN:** This block imports the modules needed by the rest of the file, including `pytest`, `sglang.srt.layers.attention.mamba.ops`, `sglang.srt.layers.attention.mamba.ops.ssu_dispatch`, `sglang.srt.server_args`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `pytest`, `sglang.srt.layers.attention.mamba.ops`, `sglang.srt.layers.attention.mamba.ops.ssu_dispatch`, `sglang.srt.server_args`。

### Lines 10-19: function init mamba ssu backend / 函数 init mamba ssu backend
```python
@pytest.fixture(scope="session", autouse=True)
def _init_mamba_ssu_backend():
    """Initialize the Mamba SSU dispatch backend for the test session.

    In production this happens in Scheduler.init_mamba_backend(). Tests have no
    scheduler, so we do it here via the same public API.
    """
    initialize_mamba_selective_state_update_backend(ServerArgs(model_path="dummy"))
    yield
    ssu_dispatch._mamba_ssu_backend = None
```
**EN:** Initialize the Mamba SSU dispatch backend for the test session. This block implements `_init_mamba_ssu_backend` and captures one focused piece of the module's behavior.
**CN:** Initialize the Mamba SSU dispatch backend for the test session. 该代码块实现 `_init_mamba_ssu_backend`，承担模块行为中的一个聚焦逻辑片段。

## Key Concepts / 关键概念
- `_init_mamba_ssu_backend`: Initialize the Mamba SSU dispatch backend for the test session. / 该代码块实现 `_init_mamba_ssu_backend`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Third-party modules / 第三方模块**: `pytest`
- **Internal modules / 内部模块**: `sglang.srt.layers.attention.mamba.ops`, `sglang.srt.layers.attention.mamba.ops.ssu_dispatch`, `sglang.srt.server_args`

- **Total lines / 总行数**: 19
