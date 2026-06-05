# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/conftest.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared pytest fixtures and hooks for pooling or embedding behavior. / [CN] 为池化或嵌入行为提供共享的 pytest fixture 与钩子。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L9)
```python
import warnings

import torch

from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `warnings`, third-party packages like `torch`, project helpers such as `vllm.platforms.current_platform`.
**CN:** 导入标准库模块（如 `warnings`）、第三方包（如 `torch`）、项目内辅助模块（如 `vllm.platforms.current_platform`）。

### Helper / 辅助函数: pytest_collection_modifyitems (L12-L28)
```python
def pytest_collection_modifyitems(config, items):
    """Configure ROCm-specific settings based on collected tests."""
    if not current_platform.is_rocm():
        return

    # Disable Flash/MemEfficient SDP on ROCm to avoid HF Transformers
    # accuracy issues: https://github.com/vllm-project/vllm/issues/30167
    # TODO: Remove once ROCm SDP accuracy issues are resolved on HuggingFace
    torch.backends.cuda.enable_flash_sdp(False)
    torch.backends.cuda.enable_mem_efficient_sdp(False)
    torch.backends.cuda.enable_math_sdp(True)
    warnings.warn(
        "ROCm: Disabled flash_sdp and mem_efficient_sdp, enabled math_sdp "
        "to avoid HuggingFace Transformers accuracy issues",
        UserWarning,
        stacklevel=1,
    )
```
**EN:** This helper encapsulates reusable logic in `pytest_collection_modifyitems`. Key inputs are `config`, `items`.
**CN:** 这个辅助函数将可复用逻辑封装在 `pytest_collection_modifyitems` 中。 关键输入包括 `config`、`items`。

## Key Concepts / 关键概念
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `warnings`
- **Third-party / 第三方**: `torch`
- **Project / 项目内**: `vllm.platforms.current_platform`
