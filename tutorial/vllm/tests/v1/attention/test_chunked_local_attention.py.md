# test_chunked_local_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/attention/test_chunked_local_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `chunked local attention` behavior and regressions in the v1 stack. / 验证 v1 栈中 `chunked local 注意力` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-11)
```python
from dataclasses import dataclass

import numpy as np
import pytest
import torch

from tests.v1.attention.utils import BatchSpec, create_common_attn_metadata
from vllm.platforms import current_platform
from vllm.v1.attention.backends.utils import make_local_attention_virtual_batches
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest, torch`. vLLM modules under test include `vllm.platforms, vllm.v1.attention.backends.utils`. Local helpers come from `tests.v1.attention.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest, torch`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm.v1.attention.backends.utils`。 本地测试辅助逻辑来自 `tests.v1.attention.utils`。

### LocalAttentionTestData (lines 15-23)
```python
class LocalAttentionTestData:
    # Input parameters
    batch_spec: BatchSpec
    attn_chunk_size: int
    block_size: int
    # Expected return values
    expected_q_seqlens: list[int]
    expected_k_seqlens: list[int]
    expected_local_block_table: list[list[int]]
```
**EN:** Class `LocalAttentionTestData` groups 0 test method(s).
**CN:** 类 `LocalAttentionTestData` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 26-152)
```python
DEVICE_TYPE = current_platform.device_type

test_data_list = [
    # Same as example in docstring of make_local_attention_virtual_batches
    # except block table has 9 columns instead of 10
    LocalAttentionTestData(
        batch_spec=BatchSpec(
            query_lens=[4, 10, 5],
            seq_lens=[6, 17, 9],
        ),
        attn_chunk_size=4,
        block_size=2,
        expected_q_seqlens=[2, 2, 1, 4, 4, 1, 4, 1],
        expected_k_seqlens=[4, 2, 4, 4, 4, 1, 4, 1],
        # 2 pages per local branch
        # (chunk size 4 // block size 2)
        expected_local_block_table=[
            [0, 1],  # local-batch 0, (batch 0, starting from k[0])
# ... excerpt omitted for brevity ...
        expected_q_seqlens=[4, 4],
        expected_k_seqlens=[4, 4],
        expected_q_seqlens=[1, 4, 2],
        expected_k_seqlens=[4, 4, 2],
        expected_k_seqlens=[1],
            [2, 2],
        ],
    ),
]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DEVICE_TYPE, test_data_list`. Shared setup calls include `LocalAttentionTestData, BatchSpec`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DEVICE_TYPE, test_data_list`。 共享初始化调用包括 `LocalAttentionTestData, BatchSpec`。

### test_local_attention_virtual_batches (lines 156-204)
```python
def test_local_attention_virtual_batches(test_data: LocalAttentionTestData):
    device = torch.device(f"{DEVICE_TYPE}:0")
    batch_spec = test_data.batch_spec
    attn_chunk_size = test_data.attn_chunk_size
    block_size = test_data.block_size
    expected_q_seqlens = test_data.expected_q_seqlens
    expected_k_seqlens = test_data.expected_k_seqlens
    expected_local_block_table = test_data.expected_local_block_table

    # Create common attention metadata
    common_attn_metadata = create_common_attn_metadata(
        batch_spec,
        block_size,
        device,
        # Use torch.arange instead of torch.randint so we can assert on
        # block table tensor values. The block table will have shape
        # (num_batches, cdiv(max_seq_len, block_size)) and the values will be
        # arranged from 0 to cdiv(max_seq_len, block_size)-1
    # ... excerpt omitted for brevity ...
    assert all(q_len <= attn_chunk_size for q_len in actual_q_seqlens)
    assert all(k_len <= attn_chunk_size for k_len in actual_k_seqlens)
    assert sum(actual_q_seqlens) == sum(batch_spec.query_lens)
    np.testing.assert_array_equal(actual_q_seqlens, expected_q_seqlens)
    np.testing.assert_array_equal(actual_k_seqlens, expected_k_seqlens)
    expected_block_table_tensor = torch.tensor(
    )
    print(f"Expected block table:\n{expected_block_table_tensor}")
    print(f"Actual block table:\n{result.block_table_tensor}")
    torch.testing.assert_close(result.block_table_tensor, expected_block_table_tensor)
```
**EN:** Parameterized test covering `local attention virtual batches`. Parameter axes: `test_data`. Inputs/fixtures: `test_data`. It exercises `mark.parametrize, torch.device, create_common_attn_metadata, make_local_attention_virtual_batches, np.diff, seq_lens_cpu.numpy`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `local 注意力 virtual batches` 的测试用例。 参数维度：`test_data`。 输入或 fixture：`test_data`。 该测试会调用 `mark.parametrize, torch.device, create_common_attn_metadata, make_local_attention_virtual_batches, np.diff, seq_lens_cpu.numpy`。 代码主体包含 3 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest, torch`.
- **CN:** 外部库：`numpy, pytest, torch`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm.v1.attention.backends.utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm.v1.attention.backends.utils`。
- **EN:** Local test helpers: `tests.v1.attention.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.attention.utils`。
- **EN:** Standard-library support: `dataclasses`.
- **CN:** 标准库支持：`dataclasses`。
