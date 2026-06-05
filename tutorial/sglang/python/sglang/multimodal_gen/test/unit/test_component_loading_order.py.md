# test_component_loading_order.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_component_loading_order.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates component loading order with focused assertions and fixtures. Key symbols include `_spec`, `_write_safetensors`, `test_component_load_order_prioritizes_weight_heavy_components`. / 该测试模块通过有针对性的断言与夹具，验证 component loading order 的实现。 关键符号包括 `_spec`, `_write_safetensors`, `test_component_load_order_prioritizes_weight_heavy_components`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and module setup / 导入与模块初始化
```python
import json

from sglang.multimodal_gen.runtime.managers.memory_managers.component_loading_order import (
    ComponentLoadSpec,
    component_load_risk_rank,
    infer_component_weight_size_bytes,
    order_component_load_specs,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 11-21: Function `_spec` / 函数 `_spec`
```python
def _spec(
    component_name: str, index: int, component_model_path: str = "/missing"
) -> ComponentLoadSpec:
    return ComponentLoadSpec(
        module_name=component_name,
        load_module_name=component_name,
        component_model_path=component_model_path,
        transformers_or_diffusers="diffusers",
        architecture=None,
        index=index,
    )
```
**EN:** This function drives `_spec` with inputs such as `component_name`, `index`, `component_model_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_spec`，主要处理 `component_name`, `index`, `component_model_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 24-35: Function `_write_safetensors` / 函数 `_write_safetensors`
```python
def _write_safetensors(path, payload_size: int) -> None:
    header = {
        "weight": {
            "dtype": "F16",
            "shape": [payload_size // 2],
            "data_offsets": [0, payload_size],
        }
    }
    header_bytes = json.dumps(header).encode("utf-8")
    path.write_bytes(
        len(header_bytes).to_bytes(8, "little") + header_bytes + b"\0" * payload_size
    )
```
**EN:** This function drives `_write_safetensors` with inputs such as `path`, `payload_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_write_safetensors`，主要处理 `path`, `payload_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 38-55: Function `test_component_load_order_prioritizes_weight_heavy_components` / 函数 `test_component_load_order_prioritizes_weight_heavy_components`
```python
def test_component_load_order_prioritizes_weight_heavy_components():
    specs = [
        _spec("scheduler", 0),
        _spec("tokenizer", 1),
        _spec("text_encoder", 2),
        _spec("transformer", 3),
        _spec("vae", 4),
    ]

    ordered_names = [spec.module_name for spec in order_component_load_specs(specs)]

    assert ordered_names == [
        "transformer",
        "text_encoder",
        "vae",
        "scheduler",
        "tokenizer",
    ]
```
**EN:** This function drives `test_component_load_order_prioritizes_weight_heavy_components`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_component_load_order_prioritizes_weight_heavy_components`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 58-75: Function `test_component_load_order_prioritizes_larger_numbered_variants` / 函数 `test_component_load_order_prioritizes_larger_numbered_variants`
```python
def test_component_load_order_prioritizes_larger_numbered_variants():
    specs = [
        _spec("transformer", 0),
        _spec("transformer_2", 1),
        _spec("text_encoder", 2),
        _spec("text_encoder_3", 3),
        _spec("text_encoder_2", 4),
    ]

    ordered_names = [spec.module_name for spec in order_component_load_specs(specs)]

    assert ordered_names == [
        "transformer_2",
        "transformer",
        "text_encoder_3",
        "text_encoder_2",
        "text_encoder",
    ]
```
**EN:** This function drives `test_component_load_order_prioritizes_larger_numbered_variants`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_component_load_order_prioritizes_larger_numbered_variants`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 78-100: Function `test_component_load_order_uses_load_module_name_for_extra_config_alias` / 函数 `test_component_load_order_uses_load_module_name_for_extra_config_alias`
```python
def test_component_load_order_uses_load_module_name_for_extra_config_alias():
    specs = [
        ComponentLoadSpec(
            module_name="condition_image_encoder",
            load_module_name="condition_image_encoder",
            component_model_path="/missing",
            transformers_or_diffusers="diffusers",
            architecture=None,
            index=0,
        ),
        ComponentLoadSpec(
            module_name="encoder_alias",
            load_module_name="text_encoder_2",
            component_model_path="/missing",
# ...

    ordered_names = [spec.module_name for spec in order_component_load_specs(specs)]

    assert ordered_names == ["encoder_alias", "condition_image_encoder"]
```
**EN:** This function drives `test_component_load_order_uses_load_module_name_for_extra_config_alias`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_component_load_order_uses_load_module_name_for_extra_config_alias`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 103-110: Function `test_component_load_risk_rank_keeps_small_helpers_last` / 函数 `test_component_load_risk_rank_keeps_small_helpers_last`
```python
def test_component_load_risk_rank_keeps_small_helpers_last():
    assert component_load_risk_rank("transformer") < component_load_risk_rank(
        "scheduler"
    )
    assert component_load_risk_rank("text_encoder_2") < component_load_risk_rank(
        "processor"
    )
    assert component_load_risk_rank("vae") < component_load_risk_rank("tokenizer")
```
**EN:** This function drives `test_component_load_risk_rank_keeps_small_helpers_last`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_component_load_risk_rank_keeps_small_helpers_last`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 113-130: Function `test_component_load_order_prefers_inferred_safetensors_size` / 函数 `test_component_load_order_prefers_inferred_safetensors_size`
```python
def test_component_load_order_prefers_inferred_safetensors_size(tmp_path):
    small_transformer_path = tmp_path / "small_transformer"
    large_encoder_path = tmp_path / "large_encoder"
    small_transformer_path.mkdir()
    large_encoder_path.mkdir()
    _write_safetensors(small_transformer_path / "model.safetensors", 16)
    _write_safetensors(large_encoder_path / "model.safetensors", 64)

    specs = [
        _spec("transformer", 0, str(small_transformer_path)),
        _spec("text_encoder", 1, str(large_encoder_path)),
        _spec("scheduler", 2),
    ]

    ordered_names = [spec.module_name for spec in order_component_load_specs(specs)]

    assert ordered_names == ["text_encoder", "transformer", "scheduler"]
    assert infer_component_weight_size_bytes(str(large_encoder_path)) == 64
```
**EN:** This function drives `test_component_load_order_prefers_inferred_safetensors_size` with inputs such as `tmp_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_component_load_order_prefers_inferred_safetensors_size`，主要处理 `tmp_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Scheduling and batching / 调度与批处理
- Automated verification / 自动化验证
- Image generation flow / 图像生成流程
- Symbol `_spec` anchors the module API / 符号 `_spec` 构成该模块的核心 API
- Symbol `_write_safetensors` anchors the module API / 符号 `_write_safetensors` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.managers.memory_managers.component_loading_order`
- **Stdlib / 标准库**: `json`
