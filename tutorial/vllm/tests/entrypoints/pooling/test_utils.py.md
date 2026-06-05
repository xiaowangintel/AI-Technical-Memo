# test_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/test_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 3 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 3 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L14)
```python
import importlib
import importlib.util
import json
import warnings
from types import SimpleNamespace

import numpy as np
import pytest
import torch

from vllm.entrypoints.pooling.utils import encode_pooling_output_float_or_ndarray
```
**EN:** Imports standard-library modules such as `importlib`, `importlib.util`, `json`, third-party packages like `fastapi.responses.ORJSONResponse`, `numpy`, `pytest`, project helpers such as `vllm.entrypoints.pooling.utils.encode_pooling_output_float_or_ndarray`.
**CN:** 导入标准库模块（如 `importlib`、`importlib.util`、`json`）、第三方包（如 `fastapi.responses.ORJSONResponse`、`numpy`、`pytest`）、项目内辅助模块（如 `vllm.entrypoints.pooling.utils.encode_pooling_output_float_or_ndarray`）。

### Helper / 辅助函数: _pooling_output (L17-L18)
```python
def _pooling_output(data):
    return SimpleNamespace(outputs=SimpleNamespace(data=data))
```
**EN:** This helper encapsulates reusable logic in `_pooling_output`. Key inputs are `data`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_pooling_output` 中。 关键输入包括 `data`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_encode_pooling_output_float_or_ndarray_returns_numpy_array (L21-L27)
```python
def test_encode_pooling_output_float_or_ndarray_returns_numpy_array():
    output = _pooling_output(torch.tensor([1.0, 2.0, 3.0], dtype=torch.float32))

    encoded = encode_pooling_output_float_or_ndarray(output)

    assert isinstance(encoded, np.ndarray)
    np.testing.assert_allclose(encoded, [1.0, 2.0, 3.0])
```
**EN:** This test validates `test_encode_pooling_output_float_or_ndarray_returns_numpy_array`. The main assertion is `isinstance(encoded, np.ndarray)`.
**CN:** 这个测试验证 `test_encode_pooling_output_float_or_ndarray_returns_numpy_array`。 核心断言是 `isinstance(encoded, np.ndarray)`。

### Test / 测试: test_orjson_serializes_numpy_array (L30-L43)
```python
@pytest.mark.skipif(
    importlib.util.find_spec("orjson") is None,
    reason="orjson is not installed",
)
def test_orjson_serializes_numpy_array():
    from fastapi.responses import ORJSONResponse

    output = _pooling_output(torch.tensor([1.0, 2.0, 3.0], dtype=torch.float32))
    encoded = encode_pooling_output_float_or_ndarray(output)

    with warnings.catch_warnings():
        warnings.simplefilter("ignore", DeprecationWarning)
        response = ORJSONResponse(content={"embedding": encoded})
    assert json.loads(response.body)["embedding"] == pytest.approx([1.0, 2.0, 3.0])
```
**EN:** This test validates `test_orjson_serializes_numpy_array`. Relevant pytest markers include `skipif`. The main assertion is `json.loads(response.body)['embedding'] == pytest.approx([1.0, 2.0, 3.0])`.
**CN:** 这个测试验证 `test_orjson_serializes_numpy_array`。 相关的 pytest 标记包括 `skipif`。 核心断言是 `json.loads(response.body)['embedding'] == pytest.approx([1.0, 2.0, 3.0])`。

### Test / 测试: test_encode_pooling_output_float_or_ndarray_falls_back_to_list (L46-L59)
```python
def test_encode_pooling_output_float_or_ndarray_falls_back_to_list():
    class DataWithUnsupportedNumpy:
        def is_contiguous(self):
            return True

        def numpy(self):
            raise TypeError("unsupported dtype")

        def tolist(self):
            return [1.0, 2.0, 3.0]

    output = _pooling_output(DataWithUnsupportedNumpy())

    assert encode_pooling_output_float_or_ndarray(output) == [1.0, 2.0, 3.0]
```
**EN:** This test validates `test_encode_pooling_output_float_or_ndarray_falls_back_to_list`. The main assertion is `encode_pooling_output_float_or_ndarray(output) == [1.0, 2.0, 3.0]`.
**CN:** 这个测试验证 `test_encode_pooling_output_float_or_ndarray_falls_back_to_list`。 核心断言是 `encode_pooling_output_float_or_ndarray(output) == [1.0, 2.0, 3.0]`。

## Key Concepts / 关键概念
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `importlib`, `importlib.util`, `json`, `types.SimpleNamespace`, `warnings`
- **Third-party / 第三方**: `fastapi.responses.ORJSONResponse`, `numpy`, `pytest`, `torch`
- **Project / 项目内**: `vllm.entrypoints.pooling.utils.encode_pooling_output_float_or_ndarray`
