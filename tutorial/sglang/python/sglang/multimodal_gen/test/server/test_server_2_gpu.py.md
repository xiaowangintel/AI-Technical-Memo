# test_server_2_gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/test_server_2_gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates server 2 gpu with focused assertions and fixtures. Key symbols include `TestDiffusionServerTwoGpu`. / 该测试模块通过有针对性的断言与夹具，验证 server 2 gpu 的实现。 关键符号包括 `TestDiffusionServerTwoGpu`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and module setup / 导入与模块初始化
```python
"""
2 GPU tests
"""

from __future__ import annotations

import pytest

from sglang.multimodal_gen.test.server.gpu_cases import TWO_GPU_CASES
from sglang.multimodal_gen.test.server.test_server_common import (  # noqa: F401
    DiffusionServerBase,
    diffusion_server,
)
from sglang.multimodal_gen.test.server.testcase_configs import DiffusionTestCase
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 17-23: Class `TestDiffusionServerTwoGpu` / 类 `TestDiffusionServerTwoGpu`
```python
class TestDiffusionServerTwoGpu(DiffusionServerBase):
    """Performance tests for 2-GPU diffusion cases."""

    @pytest.fixture(params=TWO_GPU_CASES, ids=lambda c: c.id)
    def case(self, request) -> DiffusionTestCase:
        """Provide a DiffusionTestCase for each 2-GPU test."""
        return request.param
```
**EN:** This class models `TestDiffusionServerTwoGpu` as a specialization of `DiffusionServerBase`. Performance tests for 2-GPU diffusion cases. Important methods include `case`.
**CN:** 该类实现 `TestDiffusionServerTwoGpu`，并继承/扩展 `DiffusionServerBase`。 文档字符串指出：Performance tests for 2-GPU diffusion cases. 其中较重要的方法包括 `case`。

## Key Concepts / 关键概念
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言
- Symbol `TestDiffusionServerTwoGpu` anchors the module API / 符号 `TestDiffusionServerTwoGpu` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.test.server.gpu_cases`, `sglang.multimodal_gen.test.server.test_server_common`, `sglang.multimodal_gen.test.server.testcase_configs`
- **External / 外部**: `__future__`, `pytest`
