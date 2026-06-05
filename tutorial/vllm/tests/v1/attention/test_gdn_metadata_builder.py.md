# test_gdn_metadata_builder.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/attention/test_gdn_metadata_builder.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for GDNAttentionMetadataBuilder.build() — specifically the reclassification of non-spec decodes as prefills when spec decodes exist. / 该文件的文档字符串表明其用途：`tests for gdnattentionmetadatabuilder.build() — specifically the reclassification of non-spec decodes as prefills when spec decodes exist`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-6)
```python
"""Tests for GDNAttentionMetadataBuilder.build() — specifically the
reclassification of non-spec decodes as prefills when spec decodes exist.
Covers the fix for https://github.com/vllm-project/vllm/issues/34845.
"""
```
**EN:** Module docstring that declares the scope of the file: Tests for GDNAttentionMetadataBuilder.build() — specifically the reclassification of non-spec decodes as prefills when spec decodes exist.
**CN:** 模块文档字符串直接说明了文件范围：`tests for gdnattentionmetadatabuilder.build() — specifically the reclassification of non-spec decodes as prefills when spec decodes exist`。

### Imports and setup / 导入与设置 (lines 8-23)
```python
from dataclasses import dataclass

import pytest
import torch

from tests.v1.attention.utils import (
    BatchSpec,
    create_common_attn_metadata,
    create_vllm_config,
)
from vllm.config import SpeculativeConfig
from vllm.v1.attention.backends.gdn_attn import (
    GDNAttentionMetadata,
    GDNAttentionMetadataBuilder,
)
from vllm.v1.kv_cache_interface import MambaSpec
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.config, vllm.v1.attention.backends.gdn_attn, vllm.v1.kv_cache_interface`. Local helpers come from `tests.v1.attention.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.config, vllm.v1.attention.backends.gdn_attn, vllm.v1.kv_cache_interface`。 本地测试辅助逻辑来自 `tests.v1.attention.utils`。

### Module state / 模块级状态 (lines 25-26)
```python
BLOCK_SIZE = 16
DEVICE = torch.device("cpu")
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `BLOCK_SIZE, DEVICE`. Shared setup calls include `torch.device`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`BLOCK_SIZE, DEVICE`。 共享初始化调用包括 `torch.device`。

### GDNBuildTestCase (lines 30-40)
```python
class GDNBuildTestCase:
    """Specification for a GDN metadata builder classification test."""

    seq_lens: list[int]
    query_lens: list[int]
    num_decode_draft_tokens: list[int] | None  # None = no spec config
    num_speculative_tokens: int
    expected_num_decodes: int
    expected_num_prefills: int
    expected_num_prefill_tokens: int
    expected_num_spec_decodes: int
```
**EN:** Class `GDNBuildTestCase` groups 0 test method(s).
**CN:** 类 `GDNBuildTestCase` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 43-121)
```python
GDN_BUILD_TEST_CASES = {
    # The original #34845 crash: non-spec query_len=1 + spec decode
    "mixed_decode_and_spec_decode": GDNBuildTestCase(
        seq_lens=[65, 20],
        query_lens=[1, 3],
        num_decode_draft_tokens=[-1, 2],
        num_speculative_tokens=2,
        expected_num_decodes=0,
        expected_num_prefills=1,
        expected_num_prefill_tokens=1,
        expected_num_spec_decodes=1,
    ),
    # All requests are spec decodes — no reclassification needed
    "pure_spec_decode": GDNBuildTestCase(
        seq_lens=[50, 30],
        query_lens=[3, 3],
        num_decode_draft_tokens=[2, 2],
    # ... excerpt omitted for brevity ...
        expected_num_prefills=0,
        expected_num_prefill_tokens=0,
        expected_num_spec_decodes=2,
        expected_num_decodes=3,
}
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `GDN_BUILD_TEST_CASES`. Shared setup calls include `GDNBuildTestCase`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`GDN_BUILD_TEST_CASES`。 共享初始化调用包括 `GDNBuildTestCase`。

### _create_gdn_builder (lines 124-144)
```python
def _create_gdn_builder(
    num_speculative_tokens: int = 0,
) -> GDNAttentionMetadataBuilder:
    """Create a GDNAttentionMetadataBuilder with minimal config."""
    vllm_config = create_vllm_config(block_size=BLOCK_SIZE)
    if num_speculative_tokens > 0:
        vllm_config.speculative_config = SpeculativeConfig(
            method="ngram",
            num_speculative_tokens=num_speculative_tokens,
        )
    mamba_spec = MambaSpec(
        block_size=BLOCK_SIZE,
        shapes=((16, 64),),
        dtypes=(torch.float16,),
    )
    return GDNAttentionMetadataBuilder(
        kv_cache_spec=mamba_spec,
        layer_names=["layer.0"],
        vllm_config=vllm_config,
        device=DEVICE,
    )
```
**EN:** Helper function `_create_gdn_builder` encapsulates reusable logic for `GDN builder`. Inputs: `num_speculative_tokens`. Key calls include `create_vllm_config, MambaSpec, GDNAttentionMetadataBuilder, SpeculativeConfig`.
**CN:** 辅助函数 `_create_gdn_builder` 封装了与 `gdn builder` 相关的可复用逻辑。 输入参数：`num_speculative_tokens`。 关键调用包括 `create_vllm_config, MambaSpec, GDNAttentionMetadataBuilder, SpeculativeConfig`。

### _build (lines 147-162)
```python
def _build(
    builder: GDNAttentionMetadataBuilder,
    batch_spec: BatchSpec,
    num_decode_draft_tokens: list[int] | None = None,
) -> GDNAttentionMetadata:
    """Build GDN attention metadata, optionally with spec-decode kwargs."""
    common = create_common_attn_metadata(batch_spec, BLOCK_SIZE, DEVICE)
    kwargs: dict = {}
    if num_decode_draft_tokens is not None:
        kwargs["num_decode_draft_tokens_cpu"] = torch.tensor(
            num_decode_draft_tokens, dtype=torch.int32
        )
        kwargs["num_accepted_tokens"] = torch.ones(
            batch_spec.batch_size, dtype=torch.int32, device=DEVICE
        )
    return builder.build(common_prefix_len=0, common_attn_metadata=common, **kwargs)
```
**EN:** Helper function `_build` encapsulates reusable logic for `build`. Inputs: `builder, batch_spec, num_decode_draft_tokens`. Key calls include `create_common_attn_metadata, builder.build, torch.tensor, torch.ones`.
**CN:** 辅助函数 `_build` 封装了与 `build` 相关的可复用逻辑。 输入参数：`builder, batch_spec, num_decode_draft_tokens`。 关键调用包括 `create_common_attn_metadata, builder.build, torch.tensor, torch.ones`。

### test_gdn_build_classification (lines 168-177)
```python
def test_gdn_build_classification(test_case: GDNBuildTestCase):
    """Test that GDN metadata builder classifies requests correctly."""
    builder = _create_gdn_builder(test_case.num_speculative_tokens)
    batch = BatchSpec(seq_lens=test_case.seq_lens, query_lens=test_case.query_lens)
    meta = _build(builder, batch, test_case.num_decode_draft_tokens)

    assert meta.num_decodes == test_case.expected_num_decodes
    assert meta.num_prefills == test_case.expected_num_prefills
    assert meta.num_prefill_tokens == test_case.expected_num_prefill_tokens
    assert meta.num_spec_decodes == test_case.expected_num_spec_decodes
```
**EN:** Parameterized test covering `GDN build classification`. Parameter axes: `test_case`. Inputs/fixtures: `test_case`. It exercises `mark.parametrize, _create_gdn_builder, BatchSpec, _build, GDN_BUILD_TEST_CASES.values, GDN_BUILD_TEST_CASES.keys`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `gdn build classification` 的测试用例。 参数维度：`test_case`。 输入或 fixture：`test_case`。 该测试会调用 `mark.parametrize, _create_gdn_builder, BatchSpec, _build, GDN_BUILD_TEST_CASES.values, GDN_BUILD_TEST_CASES.keys`。 代码主体包含 4 个显式断言。

### test_has_initial_state_after_reclassification (lines 180-191)
```python
def test_has_initial_state_after_reclassification():
    """After reclassification, num_prefills > 0 so the prefill kernel path
    should compute has_initial_state. For the reclassified request with
    context_lens > 0, the corresponding entry must be True."""
    builder = _create_gdn_builder(num_speculative_tokens=2)
    batch = BatchSpec(seq_lens=[65, 20], query_lens=[1, 3])
    meta = _build(builder, batch, num_decode_draft_tokens=[-1, 2])

    assert meta.num_prefills > 0, "reclassification should produce prefills"
    assert meta.has_initial_state is not None
    # req0 has context_lens = 65 - 1 = 64 > 0, so has_initial_state[0] = True
    assert meta.has_initial_state[0].item() is True
```
**EN:** Test case covering `has initial state after reclassification`. It exercises `_create_gdn_builder, BatchSpec, _build, has_initial_state.item`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `has initial state after reclassification` 的测试用例。 该测试会调用 `_create_gdn_builder, BatchSpec, _build, has_initial_state.item`。 代码主体包含 3 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.config, vllm.v1.attention.backends.gdn_attn, vllm.v1.kv_cache_interface`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.v1.attention.backends.gdn_attn, vllm.v1.kv_cache_interface`。
- **EN:** Local test helpers: `tests.v1.attention.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.attention.utils`。
- **EN:** Standard-library support: `dataclasses`.
- **CN:** 标准库支持：`dataclasses`。
