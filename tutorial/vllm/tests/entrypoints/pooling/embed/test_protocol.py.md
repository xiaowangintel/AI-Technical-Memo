# test_protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_protocol.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 15 test(s), 1 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 15 个测试、1 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L6-L14)
```python
import struct

import numpy as np
import pybase64 as base64
import pytest

from vllm.entrypoints.pooling.embed.protocol import (
    build_typed_embeddings,
)
```
**EN:** Imports standard-library modules such as `struct`, third-party packages like `numpy`, `pybase64`, `pytest`, project helpers such as `vllm.entrypoints.pooling.embed.protocol.build_typed_embeddings`.
**CN:** 导入标准库模块（如 `struct`）、第三方包（如 `numpy`、`pybase64`、`pytest`）、项目内辅助模块（如 `vllm.entrypoints.pooling.embed.protocol.build_typed_embeddings`）。

### Fixture / 夹具: sample_embeddings (L17-L22)
```python
@pytest.fixture
def sample_embeddings() -> list[list[float]]:
    return [
        [0.1, -0.2, 0.3, -0.4, 0.5, -0.6, 0.7, -0.8],
        [-0.05, 0.15, -0.25, 0.35, -0.45, 0.55, -0.65, 0.75],
    ]
```
**EN:** This fixture prepares `sample_embeddings` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `sample_embeddings`。

### Class / 类: TestBuildTypedEmbeddingsFloat (L25-L33)
```python
class TestBuildTypedEmbeddingsFloat:
    def test_float_passthrough(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["float"])
        assert result.float == sample_embeddings
        assert result.binary is None

    def test_empty_input(self):
        result = build_typed_embeddings([], ["float"])
        assert result.float == []
```
**EN:** This class groups related scenarios in `TestBuildTypedEmbeddingsFloat`. It contains 2 test method(s) and 0 supporting method(s). Representative methods include `test_float_passthrough`, `test_empty_input`.
**CN:** 该类将与 `TestBuildTypedEmbeddingsFloat` 相关的场景组织在一起。 它包含 2 个测试方法和 0 个辅助方法。 代表性方法包括 `test_float_passthrough`、`test_empty_input`。

### Test method / 测试方法: TestBuildTypedEmbeddingsFloat.test_float_passthrough (L26-L29)
```python
    def test_float_passthrough(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["float"])
        assert result.float == sample_embeddings
        assert result.binary is None
```
**EN:** This test validates `TestBuildTypedEmbeddingsFloat.test_float_passthrough`. Key inputs are `sample_embeddings`. The main assertion is `result.float == sample_embeddings` and `result.binary is None`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsFloat.test_float_passthrough`。 关键输入包括 `sample_embeddings`。 核心断言是 `result.float == sample_embeddings` and `result.binary is None`。

### Test method / 测试方法: TestBuildTypedEmbeddingsFloat.test_empty_input (L31-L33)
```python
    def test_empty_input(self):
        result = build_typed_embeddings([], ["float"])
        assert result.float == []
```
**EN:** This test validates `TestBuildTypedEmbeddingsFloat.test_empty_input`. The main assertion is `result.float == []`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsFloat.test_empty_input`。 核心断言是 `result.float == []`。

### Class / 类: TestBuildTypedEmbeddingsBinary (L36-L86)
```python
class TestBuildTypedEmbeddingsBinary:
    def test_binary_packing(self):
        # 8 values: positive->1, negative->0 => bits: 10101010 = 0xAA = 170
        # signed: 170 - 128 = 42
        embs = [[1.0, -1.0, 1.0, -1.0, 1.0, -1.0, 1.0, -1.0]]
        result = build_typed_embeddings(embs, ["binary"])
        assert result.binary is not None
        assert result.binary[0] == [42]

    def test_ubinary_packing(self):
        embs = [[1.0, -1.0, 1.0, -1.0, 1.0, -1.0, 1.0, -1.0]]
        result = build_typed_embeddings(embs, ["ubinary"])
        assert result.ubinary is not None
        assert result.ubinary[0] == [170]  # 0b10101010
# ... 29 lines omitted for brevity ...
        embs = [[0.1] * 7]
        with pytest.raises(ValueError, match="multiple of 8"):
            build_typed_embeddings(embs, ["binary"])

    def test_ubinary_non_multiple_of_8_raises(self):
        embs = [[0.1] * 10]
        with pytest.raises(ValueError, match="multiple of 8"):
            build_typed_embeddings(embs, ["ubinary"])
```
**EN:** This class groups related scenarios in `TestBuildTypedEmbeddingsBinary`. It contains 8 test method(s) and 0 supporting method(s). Representative methods include `test_binary_packing`, `test_ubinary_packing`, `test_binary_all_positive`.
**CN:** 该类将与 `TestBuildTypedEmbeddingsBinary` 相关的场景组织在一起。 它包含 8 个测试方法和 0 个辅助方法。 代表性方法包括 `test_binary_packing`、`test_ubinary_packing`、`test_binary_all_positive`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBinary.test_binary_packing (L37-L43)
```python
    def test_binary_packing(self):
        # 8 values: positive->1, negative->0 => bits: 10101010 = 0xAA = 170
        # signed: 170 - 128 = 42
        embs = [[1.0, -1.0, 1.0, -1.0, 1.0, -1.0, 1.0, -1.0]]
        result = build_typed_embeddings(embs, ["binary"])
        assert result.binary is not None
        assert result.binary[0] == [42]
```
**EN:** This test validates `TestBuildTypedEmbeddingsBinary.test_binary_packing`. The main assertion is `result.binary is not None` and `result.binary[0] == [42]`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBinary.test_binary_packing`。 核心断言是 `result.binary is not None` and `result.binary[0] == [42]`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBinary.test_ubinary_packing (L45-L49)
```python
    def test_ubinary_packing(self):
        embs = [[1.0, -1.0, 1.0, -1.0, 1.0, -1.0, 1.0, -1.0]]
        result = build_typed_embeddings(embs, ["ubinary"])
        assert result.ubinary is not None
        assert result.ubinary[0] == [170]  # 0b10101010
```
**EN:** This test validates `TestBuildTypedEmbeddingsBinary.test_ubinary_packing`. The main assertion is `result.ubinary is not None` and `result.ubinary[0] == [170]`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBinary.test_ubinary_packing`。 核心断言是 `result.ubinary is not None` and `result.ubinary[0] == [170]`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBinary.test_binary_all_positive (L51-L56)
```python
    def test_binary_all_positive(self):
        embs = [[0.1] * 8]
        result = build_typed_embeddings(embs, ["binary"])
        assert result.binary is not None
        # all bits = 1 => 0xFF = 255, signed: 255 - 128 = 127
        assert result.binary[0] == [127]
```
**EN:** This test validates `TestBuildTypedEmbeddingsBinary.test_binary_all_positive`. The main assertion is `result.binary is not None` and `result.binary[0] == [127]`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBinary.test_binary_all_positive`。 核心断言是 `result.binary is not None` and `result.binary[0] == [127]`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBinary.test_binary_all_negative (L58-L63)
```python
    def test_binary_all_negative(self):
        embs = [[-0.1] * 8]
        result = build_typed_embeddings(embs, ["binary"])
        assert result.binary is not None
        # all bits = 0, signed: 0 - 128 = -128
        assert result.binary[0] == [-128]
```
**EN:** This test validates `TestBuildTypedEmbeddingsBinary.test_binary_all_negative`. The main assertion is `result.binary is not None` and `result.binary[0] == [-128]`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBinary.test_binary_all_negative`。 核心断言是 `result.binary is not None` and `result.binary[0] == [-128]`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBinary.test_binary_dimension_is_eighth (L65-L69)
```python
    def test_binary_dimension_is_eighth(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["binary"])
        assert result.binary is not None
        for orig, packed in zip(sample_embeddings, result.binary):
            assert len(packed) == len(orig) // 8
```
**EN:** This test validates `TestBuildTypedEmbeddingsBinary.test_binary_dimension_is_eighth`. Key inputs are `sample_embeddings`. The main assertion is `result.binary is not None` and `len(packed) == len(orig) // 8`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBinary.test_binary_dimension_is_eighth`。 关键输入包括 `sample_embeddings`。 核心断言是 `result.binary is not None` and `len(packed) == len(orig) // 8`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBinary.test_zero_treated_as_positive (L71-L76)
```python
    def test_zero_treated_as_positive(self):
        embs = [[0.0] * 8]
        result = build_typed_embeddings(embs, ["binary"])
        assert result.binary is not None
        # 0.0 >= 0 is True, so bit=1 for all => 127 (signed)
        assert result.binary[0] == [127]
```
**EN:** This test validates `TestBuildTypedEmbeddingsBinary.test_zero_treated_as_positive`. The main assertion is `result.binary is not None` and `result.binary[0] == [127]`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBinary.test_zero_treated_as_positive`。 核心断言是 `result.binary is not None` and `result.binary[0] == [127]`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBinary.test_non_multiple_of_8_raises (L78-L81)
```python
    def test_non_multiple_of_8_raises(self):
        embs = [[0.1] * 7]
        with pytest.raises(ValueError, match="multiple of 8"):
            build_typed_embeddings(embs, ["binary"])
```
**EN:** This test validates `TestBuildTypedEmbeddingsBinary.test_non_multiple_of_8_raises`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBinary.test_non_multiple_of_8_raises`。 它使用 `pytest.raises` 检查预期失败路径。

### Test method / 测试方法: TestBuildTypedEmbeddingsBinary.test_ubinary_non_multiple_of_8_raises (L83-L86)
```python
    def test_ubinary_non_multiple_of_8_raises(self):
        embs = [[0.1] * 10]
        with pytest.raises(ValueError, match="multiple of 8"):
            build_typed_embeddings(embs, ["ubinary"])
```
**EN:** This test validates `TestBuildTypedEmbeddingsBinary.test_ubinary_non_multiple_of_8_raises`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBinary.test_ubinary_non_multiple_of_8_raises`。 它使用 `pytest.raises` 检查预期失败路径。

### Class / 类: TestBuildTypedEmbeddingsBase64 (L89-L106)
```python
class TestBuildTypedEmbeddingsBase64:
    def test_base64_roundtrip(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["base64"])
        assert result.base64 is not None
        assert len(result.base64) == 2

        for orig, b64_str in zip(sample_embeddings, result.base64):
            decoded = base64.b64decode(b64_str)
            n = len(orig)
            values = struct.unpack(f"<{n}f", decoded)
            np.testing.assert_allclose(orig, values, rtol=1e-5)

    def test_base64_byte_length(self):
        embs = [[0.1, 0.2, 0.3]]
        result = build_typed_embeddings(embs, ["base64"])
        assert result.base64 is not None
        raw = base64.b64decode(result.base64[0])
        assert len(raw) == 3 * 4  # 3 floats * 4 bytes each
```
**EN:** This class groups related scenarios in `TestBuildTypedEmbeddingsBase64`. It contains 2 test method(s) and 0 supporting method(s). Representative methods include `test_base64_roundtrip`, `test_base64_byte_length`.
**CN:** 该类将与 `TestBuildTypedEmbeddingsBase64` 相关的场景组织在一起。 它包含 2 个测试方法和 0 个辅助方法。 代表性方法包括 `test_base64_roundtrip`、`test_base64_byte_length`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBase64.test_base64_roundtrip (L90-L99)
```python
    def test_base64_roundtrip(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["base64"])
        assert result.base64 is not None
        assert len(result.base64) == 2

        for orig, b64_str in zip(sample_embeddings, result.base64):
            decoded = base64.b64decode(b64_str)
            n = len(orig)
            values = struct.unpack(f"<{n}f", decoded)
            np.testing.assert_allclose(orig, values, rtol=1e-5)
```
**EN:** This test validates `TestBuildTypedEmbeddingsBase64.test_base64_roundtrip`. Key inputs are `sample_embeddings`. The main assertion is `result.base64 is not None` and `len(result.base64) == 2`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBase64.test_base64_roundtrip`。 关键输入包括 `sample_embeddings`。 核心断言是 `result.base64 is not None` and `len(result.base64) == 2`。

### Test method / 测试方法: TestBuildTypedEmbeddingsBase64.test_base64_byte_length (L101-L106)
```python
    def test_base64_byte_length(self):
        embs = [[0.1, 0.2, 0.3]]
        result = build_typed_embeddings(embs, ["base64"])
        assert result.base64 is not None
        raw = base64.b64decode(result.base64[0])
        assert len(raw) == 3 * 4  # 3 floats * 4 bytes each
```
**EN:** This test validates `TestBuildTypedEmbeddingsBase64.test_base64_byte_length`. The main assertion is `result.base64 is not None` and `len(raw) == 3 * 4`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsBase64.test_base64_byte_length`。 核心断言是 `result.base64 is not None` and `len(raw) == 3 * 4`。

### Class / 类: TestBuildTypedEmbeddingsMultiple (L109-L129)
```python
class TestBuildTypedEmbeddingsMultiple:
    def test_all_types_at_once(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(
            sample_embeddings,
            ["float", "binary", "ubinary", "base64"],
        )
        assert result.float is not None
        assert result.binary is not None
        assert result.ubinary is not None
        assert result.base64 is not None

    def test_subset_types(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["float", "binary"])
        assert result.float is not None
        assert result.binary is not None
        assert result.ubinary is None
        assert result.base64 is None

    def test_unknown_type_ignored(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["float", "unknown_type"])
        assert result.float is not None
```
**EN:** This class groups related scenarios in `TestBuildTypedEmbeddingsMultiple`. It contains 3 test method(s) and 0 supporting method(s). Representative methods include `test_all_types_at_once`, `test_subset_types`, `test_unknown_type_ignored`.
**CN:** 该类将与 `TestBuildTypedEmbeddingsMultiple` 相关的场景组织在一起。 它包含 3 个测试方法和 0 个辅助方法。 代表性方法包括 `test_all_types_at_once`、`test_subset_types`、`test_unknown_type_ignored`。

### Test method / 测试方法: TestBuildTypedEmbeddingsMultiple.test_all_types_at_once (L110-L118)
```python
    def test_all_types_at_once(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(
            sample_embeddings,
            ["float", "binary", "ubinary", "base64"],
        )
        assert result.float is not None
        assert result.binary is not None
        assert result.ubinary is not None
        assert result.base64 is not None
```
**EN:** This test validates `TestBuildTypedEmbeddingsMultiple.test_all_types_at_once`. Key inputs are `sample_embeddings`. The main assertion is `result.float is not None` and `result.binary is not None`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsMultiple.test_all_types_at_once`。 关键输入包括 `sample_embeddings`。 核心断言是 `result.float is not None` and `result.binary is not None`。

### Test method / 测试方法: TestBuildTypedEmbeddingsMultiple.test_subset_types (L120-L125)
```python
    def test_subset_types(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["float", "binary"])
        assert result.float is not None
        assert result.binary is not None
        assert result.ubinary is None
        assert result.base64 is None
```
**EN:** This test validates `TestBuildTypedEmbeddingsMultiple.test_subset_types`. Key inputs are `sample_embeddings`. The main assertion is `result.float is not None` and `result.binary is not None`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsMultiple.test_subset_types`。 关键输入包括 `sample_embeddings`。 核心断言是 `result.float is not None` and `result.binary is not None`。

### Test method / 测试方法: TestBuildTypedEmbeddingsMultiple.test_unknown_type_ignored (L127-L129)
```python
    def test_unknown_type_ignored(self, sample_embeddings: list[list[float]]):
        result = build_typed_embeddings(sample_embeddings, ["float", "unknown_type"])
        assert result.float is not None
```
**EN:** This test validates `TestBuildTypedEmbeddingsMultiple.test_unknown_type_ignored`. Key inputs are `sample_embeddings`. The main assertion is `result.float is not None`.
**CN:** 这个测试验证 `TestBuildTypedEmbeddingsMultiple.test_unknown_type_ignored`。 关键输入包括 `sample_embeddings`。 核心断言是 `result.float is not None`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `struct`
- **Third-party / 第三方**: `numpy`, `pybase64`, `pytest`
- **Project / 项目内**: `vllm.entrypoints.pooling.embed.protocol.build_typed_embeddings`
