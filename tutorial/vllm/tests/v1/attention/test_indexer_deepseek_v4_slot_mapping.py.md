# test_indexer_deepseek_v4_slot_mapping.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/attention/test_indexer_deepseek_v4_slot_mapping.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `indexer deepseek v4 slot mapping` behavior and regressions in the v1 stack. / 验证 v1 栈中 `indexer deepseek v4 slot mapping` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-10)
```python
import pytest
import torch

from tests.v1.attention.utils import create_vllm_config
from vllm.v1.attention.backend import CommonAttentionMetadata
from vllm.v1.attention.backends.mla.indexer import DeepseekV32IndexerMetadataBuilder
from vllm.v1.kv_cache_interface import MLAAttentionSpec
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.v1.attention.backend, vllm.v1.attention.backends.mla.indexer, vllm.v1.kv_cache_interface`. Local helpers come from `tests.v1.attention.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.v1.attention.backend, vllm.v1.attention.backends.mla.indexer, vllm.v1.kv_cache_interface`。 本地测试辅助逻辑来自 `tests.v1.attention.utils`。

### test_indexer_builder_deepseek_v4_compressed_slot_mapping_uses_storage_block_size (lines 14-92)
```python
def test_indexer_builder_deepseek_v4_compressed_slot_mapping_uses_storage_block_size():
    """Regression test: DeepseekV4 compression path must compute slot_mapping from
    compressed positions, not reuse the uncompressed common metadata mapping.
    """
    device = torch.device("cuda")

    # storage_block_size = block_size // compress_ratio = 256 // 4 = 64
    kv_cache_spec = MLAAttentionSpec(
        block_size=256,
        num_kv_heads=1,
        head_size=128,
        dtype=torch.bfloat16,
        compress_ratio=4,
    )
    vllm_config = create_vllm_config(max_model_len=1024)
    builder = DeepseekV32IndexerMetadataBuilder(
        kv_cache_spec=kv_cache_spec,
        layer_names=["dummy"],
    # ... excerpt omitted for brevity ...
    assert md.slot_mapping.numel() == 40
    assert valid_slots.numel() == 10  # 40 tokens / compress_ratio 4
    expected = torch.tensor(
            7 * storage_bs + 5,
        ],
        dtype=torch.int64,
        device=device,
    torch.testing.assert_close(valid_slots, expected)
```
**EN:** Test case covering `indexer builder deepseek v4 compressed slot mapping uses storage block size`. It exercises `mark.skipif, torch.device, MLAAttentionSpec, create_vllm_config, DeepseekV32IndexerMetadataBuilder, torch.tensor`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `indexer builder deepseek v4 compressed slot mapping uses storage block size` 的测试用例。 该测试会调用 `mark.skipif, torch.device, MLAAttentionSpec, create_vllm_config, DeepseekV32IndexerMetadataBuilder, torch.tensor`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.v1.attention.backend, vllm.v1.attention.backends.mla.indexer, vllm.v1.kv_cache_interface`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.attention.backend, vllm.v1.attention.backends.mla.indexer, vllm.v1.kv_cache_interface`。
- **EN:** Local test helpers: `tests.v1.attention.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.attention.utils`。
