# accuracy_config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/accuracy_config.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates accuracy config with focused assertions and fixtures. Key symbols include `ComponentType`, `ComponentSkip`, `get_threshold`. / 该测试模块通过有针对性的断言与夹具，验证 accuracy config 的实现。 关键符号包括 `ComponentType`, `ComponentSkip`, `get_threshold`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup / 导入与模块初始化
```python
from __future__ import annotations

from dataclasses import dataclass
from enum import Enum
from typing import Dict, Optional

from sglang.multimodal_gen.test.server.testcase_configs import DiffusionTestCase
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 10-13: Class `ComponentType` / 类 `ComponentType`
```python
class ComponentType(str, Enum):
    VAE = "vae"
    TRANSFORMER = "transformer"
    TEXT_ENCODER = "text_encoder"
```
**EN:** This class models `ComponentType` as a specialization of `str`, `Enum`.
**CN:** 该类实现 `ComponentType`，并继承/扩展 `str`, `Enum`。

### Lines 16-18: Class `ComponentSkip` / 类 `ComponentSkip`
```python
@dataclass(frozen=True)
class ComponentSkip:
    reason: str
```
**EN:** This class models `ComponentSkip`.
**CN:** 该类实现 `ComponentSkip`。

### Lines 19-347: Top-level configuration / 顶层配置
```python


DEFAULT_TIMESTEP = 500.0
TIMESTEP_NORMALIZATION_FACTOR = 1000.0
I2V_IMAGE_DIM = 1280
I2V_TEXT_ENCODER_DIM = 5120

DEFAULT_TEXT_ENCODER_VOCAB_SIZE = 32000
TEXT_ENCODER_INPUT_SEED = 42
TEXT_ENCODER_TOKEN_MIN = 100
TEXT_ENCODER_TOKEN_MAX = 30000
TEXT_ENCODER_TOKEN_LENGTH = 32

# Default thresholds by component. Override per component/case if needed.
# ...
            "HF config cannot be parsed as valid JSON for component reference loading"
        ),
    },
}
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 353-355: Function `get_threshold` / 函数 `get_threshold`
```python
def get_threshold(case_id: str, component: ComponentType) -> float:
    overrides = CASE_THRESHOLDS.get(case_id, {})
    return overrides.get(component, DEFAULT_THRESHOLDS[component])
```
**EN:** This function drives `get_threshold` with inputs such as `case_id`, `component`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_threshold`，主要处理 `case_id`, `component` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 358-362: Function `get_skip_reason` / 函数 `get_skip_reason`
```python
def get_skip_reason(case: DiffusionTestCase, component: ComponentType) -> Optional[str]:
    skip_entry = SKIP_COMPONENTS.get(case.id, {}).get(component)
    if skip_entry is None:
        return None
    return skip_entry.reason
```
**EN:** This function drives `get_skip_reason` with inputs such as `case`, `component`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_skip_reason`，主要处理 `case`, `component` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 365-366: Function `should_skip_component` / 函数 `should_skip_component`
```python
def should_skip_component(case: DiffusionTestCase, component: ComponentType) -> bool:
    return get_skip_reason(case, component) is not None
```
**EN:** This function drives `should_skip_component` with inputs such as `case`, `component`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `should_skip_component`，主要处理 `case`, `component` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Model/component loading / 模型/组件加载
- Caching strategy / 缓存策略
- Automated verification / 自动化验证
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.test.server.testcase_configs`
- **External / 外部**: `__future__`
- **Stdlib / 标准库**: `dataclasses`, `enum`, `typing`
