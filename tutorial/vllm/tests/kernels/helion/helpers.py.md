# helpers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/helion/helpers.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / helion / helpers; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / helion / helpers 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-17)
```python
import json
import tempfile
from collections.abc import Callable
from contextlib import contextmanager
from pathlib import Path
from typing import Any
from unittest.mock import patch

import helion

from vllm.kernels.helion.case_key import CaseKey
from vllm.kernels.helion.config_manager import ConfigManager
from vllm.kernels.helion.register import register_kernel
from vllm.kernels.helion.utils import get_canonical_gpu_name
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as json, tempfile, collections.abc, contextlib; and vLLM components like vllm.kernels.helion.case_key, vllm.kernels.helion.config_manager, vllm.kernels.helion.register, vllm.kernels.helion.utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 json、tempfile、collections.abc、contextlib；vLLM 内部组件，例如 vllm.kernels.helion.case_key、vllm.kernels.helion.config_manager、vllm.kernels.helion.register、vllm.kernels.helion.utils。

### Constants and module state (lines 19-23)
```python
GPU_PLATFORM = get_canonical_gpu_name()

DEFAULT_CONFIGS: dict[CaseKey, helion.Config] = {
    CaseKey.default(): helion.Config(block_sizes=[32]),
}
```
**EN:** This block centralizes shared constants and parameter grids, including GPU_PLATFORM, DEFAULT_CONFIGS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 GPU_PLATFORM、DEFAULT_CONFIGS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `dummy_kernel_registry` (lines 26-75)
```python
@contextmanager
def dummy_kernel_registry(
    configs: dict[CaseKey, helion.Config] | None = None,
):
    """Context manager providing a register function with automatic config setup.

    Yields a ``register`` callable with the same signature as
    ``register_kernel``.  Before applying the real decorator it writes a
    config JSON for the kernel name (from ``op_name`` or ``fn.__name__``)
    into a temporary directory backed by a fresh ``ConfigManager``.
    """
    if configs is None:
        configs = DEFAULT_CONFIGS

    def _to_config_entries(cfgs: dict) -> list[dict[str, Any]]:
        pairs: list[dict[str, Any]] = []
        for k, v in cfgs.items():
            config_data = v.__dict__["config"]
            pairs.append({"key": dict(k), "config": config_data})
        return pairs

    with tempfile.TemporaryDirectory() as tmpdir:
        config_dir = Path(tmpdir)
        ConfigManager.reset_instance()
        cm = ConfigManager(base_dir=config_dir)

        with patch(
            "vllm.kernels.helion.config_manager.ConfigManager",
            return_value=cm,
        ):

            def register(
                op_name: str | None = None,
                **kwargs,
            ) -> Callable:
                def decorator(fn: Callable) -> Callable:
                    name = op_name or fn.__name__
                    kernel_dir = config_dir / name
                    kernel_dir.mkdir(parents=True, exist_ok=True)
                    (kernel_dir / f"{GPU_PLATFORM}.json").write_text(
                        json.dumps(_to_config_entries(configs))
                    )
                    return register_kernel(op_name, **kwargs)(fn)

                return decorator

            try:
                yield register
            finally:
                ConfigManager.reset_instance()
```
**EN:** This helper function implements the shared logic for dummy kernel registry. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 dummy kernel registry 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

## Key Concepts / 关键概念
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `json`
- `tempfile`
- `collections.abc -> Callable`
- `contextlib -> contextmanager`
- `pathlib -> Path`
- `typing -> Any`
- `unittest.mock -> patch`
- `helion`
- `vllm.kernels.helion.case_key -> CaseKey`
- `vllm.kernels.helion.config_manager -> ConfigManager`
- `vllm.kernels.helion.register -> register_kernel`
- `vllm.kernels.helion.utils -> get_canonical_gpu_name`
