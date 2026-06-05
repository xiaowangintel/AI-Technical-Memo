# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/conftest.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest fixture module for compile / conftest, providing reusable setup, factories, and environment controls. / compile / conftest 对应的 pytest fixture 模块，提供可复用的初始化、工厂函数和环境控制。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-9)
```python
from contextlib import contextmanager
from unittest.mock import MagicMock, patch

import pytest

from vllm.platforms.interface import DeviceCapability
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as contextlib, unittest.mock, pytest; and vLLM components like vllm.platforms.interface.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 contextlib、unittest.mock、pytest；vLLM 内部组件，例如 vllm.platforms.interface。

### Function `mock_cuda_platform` (lines 12-48)
```python
@pytest.fixture
def mock_cuda_platform():
    """
    Fixture that returns a factory for creating mocked CUDA platforms.

    Usage:
        def test_something(mock_cuda_platform):
            with mock_cuda_platform(is_cuda=True, capability=(9, 0)):
                # test code
    """

    @contextmanager
    def _mock_platform(is_cuda: bool = True, capability: tuple[int, int] | None = None):
        mock_platform = MagicMock()
        mock_platform.is_cuda.return_value = is_cuda
        device_capability = (
            DeviceCapability(*capability) if capability is not None else None
        )
        mock_platform.get_device_capability.return_value = device_capability

        def is_device_capability_family(
            requested_capability: int, device_id: int = 0
        ) -> bool:
            current_capability = mock_platform.get_device_capability(
                device_id=device_id
            )
            if current_capability is None:
                return False
            return current_capability.major == (requested_capability // 10)

        mock_platform.is_device_capability_family.side_effect = (
            is_device_capability_family
        )
        with patch("vllm.platforms.current_platform", mock_platform):
            yield mock_platform

    return _mock_platform
```
**EN:** This fixture prepares reusable state for mock CUDA platform. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 mock CUDA platform 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

## Key Concepts / 关键概念
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `contextlib -> contextmanager`
- `unittest.mock -> MagicMock, patch`
- `pytest`
- `vllm.platforms.interface -> DeviceCapability`
