# test_component_accuracy_2_gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/test_component_accuracy_2_gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates component accuracy 2 gpu with focused assertions and fixtures. Key symbols include `TestComponentAccuracy2GPU`. / 该测试模块通过有针对性的断言与夹具，验证 component accuracy 2 gpu 的实现。 关键符号包括 `TestComponentAccuracy2GPU`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup / 导入与模块初始化
```python
import pytest

from sglang.multimodal_gen.test.server.accuracy_config import (
    ComponentType,
    get_skip_reason,
    should_skip_component,
)
from sglang.multimodal_gen.test.server.accuracy_testcase_configs import (
    ACCURACY_TWO_GPU_CASES,
    get_component_duplicate_skip_reason,
)
from sglang.multimodal_gen.test.server.accuracy_utils import (
    run_native_component_accuracy_case,
    run_text_encoder_accuracy_case,
)
from sglang.multimodal_gen.test.server.component_accuracy import AccuracyEngine
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 19-65: Class `TestComponentAccuracy2GPU` / 类 `TestComponentAccuracy2GPU`
```python
@pytest.mark.parametrize("case", ACCURACY_TWO_GPU_CASES, ids=lambda case: case.id)
class TestComponentAccuracy2GPU:
    """2-GPU component accuracy suite."""

    def test_vae_accuracy(self, case):
        if should_skip_component(case, ComponentType.VAE):
            pytest.skip(get_skip_reason(case, ComponentType.VAE))
        duplicate_reason = get_component_duplicate_skip_reason(case, ComponentType.VAE)
        if duplicate_reason:
            pytest.skip(duplicate_reason)
        run_native_component_accuracy_case(
            AccuracyEngine,
            case,
            ComponentType.VAE,
# ...
            AccuracyEngine,
            case,
            case.server_args.num_gpus,
        )
```
**EN:** This class models `TestComponentAccuracy2GPU`. 2-GPU component accuracy suite. Important methods include `test_vae_accuracy`, `test_transformer_accuracy`, `test_encoder_accuracy`.
**CN:** 该类实现 `TestComponentAccuracy2GPU`。 文档字符串指出：2-GPU component accuracy suite. 其中较重要的方法包括 `test_vae_accuracy`, `test_transformer_accuracy`, `test_encoder_accuracy`。

## Key Concepts / 关键概念
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言
- Symbol `TestComponentAccuracy2GPU` anchors the module API / 符号 `TestComponentAccuracy2GPU` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.test.server.accuracy_config`, `sglang.multimodal_gen.test.server.accuracy_testcase_configs`, `sglang.multimodal_gen.test.server.accuracy_utils`, `sglang.multimodal_gen.test.server.component_accuracy`
- **External / 外部**: `pytest`
