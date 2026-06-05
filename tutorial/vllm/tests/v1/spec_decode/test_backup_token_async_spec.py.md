# test_backup_token_async_spec.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/spec_decode/test_backup_token_async_spec.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Regression tests for the backup token fix in prepare_next_token_ids_padded. / 该文件的文档字符串表明其用途：`regression tests for the backup token fix in prepare_next_token_ids_padded`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-7)
```python
"""Regression tests for the backup token fix in prepare_next_token_ids_padded.

Fixes #38098: with async scheduling, seq_lens_cpu is inflated by unaccepted
draft token placeholders, causing get_token_id() to return -1.
"""
```
**EN:** Module docstring that declares the scope of the file: Regression tests for the backup token fix in prepare_next_token_ids_padded.
**CN:** 模块文档字符串直接说明了文件范围：`regression tests for the backup token fix in prepare_next_token_ids_padded`。

### Imports and setup / 导入与设置 (lines 9-13)
```python
from __future__ import annotations

import numpy as np
import pytest
import torch
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest, torch`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest, torch`。

### _FakeRequest (lines 16-32)
```python
class _FakeRequest:
    def __init__(self, prompt_tokens: list[int], output_tokens: list[int]):
        self.num_prompt_tokens = len(prompt_tokens)
        self._prompt = prompt_tokens
        self._output = output_tokens

    @property
    def num_tokens(self) -> int:
        return self.num_prompt_tokens + len(self._output)

    def get_token_id(self, idx: int) -> int:
        if idx < self.num_prompt_tokens:
            return self._prompt[idx]
        out_idx = idx - self.num_prompt_tokens
        if out_idx < len(self._output):
            return self._output[out_idx]
        return -1  # out of range
```
**EN:** Class `_FakeRequest` groups 0 test method(s) and 3 helper/fixture method(s).
**CN:** 类 `_FakeRequest` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。

### _FakeInputBatch (lines 35-45)
```python
class _FakeInputBatch:
    def __init__(
        self,
        req_ids: list[str],
        num_tokens_no_spec: list[int],
        vocab_size: int = 32000,
    ):
        self.req_ids = req_ids
        self.num_reqs = len(req_ids)
        self.vocab_size = vocab_size
        self.num_tokens_no_spec = np.array(num_tokens_no_spec, dtype=np.int64)
```
**EN:** Class `_FakeInputBatch` groups 0 test method(s) and 1 helper/fixture method(s).
**CN:** 类 `_FakeInputBatch` 组织了 0 个测试方法，以及 1 个辅助或 fixture 方法。

### _make_requests (lines 48-56)
```python
def _make_requests(
    req_ids: list[str],
    prompt_lens: list[int],
    output_lens: list[int],
) -> dict[str, _FakeRequest]:
    requests = {}
    for rid, plen, olen in zip(req_ids, prompt_lens, output_lens):
        requests[rid] = _FakeRequest(list(range(plen)), list(range(1000, 1000 + olen)))
    return requests
```
**EN:** Helper function `_make_requests` encapsulates reusable logic for `requests`. Inputs: `req_ids, prompt_lens, output_lens`. Key calls include `zip, _FakeRequest, list, range`.
**CN:** 辅助函数 `_make_requests` 封装了与 `requests` 相关的可复用逻辑。 输入参数：`req_ids, prompt_lens, output_lens`。 关键调用包括 `zip, _FakeRequest, list, range`。

### _backup_buggy (lines 59-68)
```python
def _backup_buggy(
    seq_lens_cpu: torch.Tensor,
    requests: dict[str, _FakeRequest],
    batch: _FakeInputBatch,
) -> list[int]:
    """Old logic: uses seq_lens_cpu directly (may be inflated)."""
    n = batch.num_reqs
    return [
        requests[batch.req_ids[i]].get_token_id(int(seq_lens_cpu[i])) for i in range(n)
    ]
```
**EN:** Helper function `_backup_buggy` encapsulates reusable logic for `backup buggy`. Inputs: `seq_lens_cpu, requests, batch`. Key calls include `requests.get_token_id, int, range`.
**CN:** 辅助函数 `_backup_buggy` 封装了与 `backup buggy` 相关的可复用逻辑。 输入参数：`seq_lens_cpu, requests, batch`。 关键调用包括 `requests.get_token_id, int, range`。

### _backup_fixed (lines 71-78)
```python
def _backup_fixed(
    requests: dict[str, _FakeRequest],
    batch: _FakeInputBatch,
) -> list[int]:
    """New logic: uses num_tokens_no_spec - 1 (last committed token)."""
    n = batch.num_reqs
    idx = (batch.num_tokens_no_spec[:n] - 1).tolist()
    return [requests[batch.req_ids[i]].get_token_id(int(idx[i])) for i in range(n)]
```
**EN:** Helper function `_backup_fixed` encapsulates reusable logic for `backup fixed`. Inputs: `requests, batch`. Key calls include `tolist, requests.get_token_id, int, range`.
**CN:** 辅助函数 `_backup_fixed` 封装了与 `backup fixed` 相关的可复用逻辑。 输入参数：`requests, batch`。 关键调用包括 `tolist, requests.get_token_id, int, range`。

### TestBackupTokenAsyncSpec (lines 81-147)
```python
class TestBackupTokenAsyncSpec:
    def test_no_inflation_fixed_returns_last_token(self):
        req_ids = ["r0", "r1"]
        requests = _make_requests(req_ids, [3, 3], [2, 2])
        batch = _FakeInputBatch(req_ids, [5, 5])
        # idx = 5-1 = 4 → output[1] = 1001
        assert _backup_fixed(requests, batch) == [1001, 1001]

    def test_inflation_buggy_returns_placeholder(self):
        # inflated by 3 spec tokens → idx 8 is out of range
        seq_lens = torch.tensor([8, 8], dtype=torch.int64)
        assert _backup_buggy(seq_lens, requests, batch) == [-1, -1]
    def test_inflation_fixed_returns_correct_token(self):
    # ... excerpt omitted for brevity ...
        assert _backup_buggy(seq_lens, requests, batch) == [-1, -1, -1]
        assert _backup_fixed(requests, batch) == [1002, 2000, 3003]
        assert _backup_fixed(requests, batch) == [30]
    @pytest.mark.parametrize("num_spec_tokens", [1, 2, 3, 4, 5])
        assert _backup_fixed(requests, batch) == [1004]
        assert _backup_fixed(requests, batch) == [1001]
        # with inflation: still -1, fixed still correct
        seq_lens_inf = torch.tensor([8], dtype=torch.int64)
        assert _backup_buggy(seq_lens_inf, requests, batch) == [-1]
```
**EN:** Class `TestBackupTokenAsyncSpec` groups 7 test method(s). Representative scenarios: `test_no_inflation_fixed_returns_last_token, test_inflation_buggy_returns_placeholder, test_inflation_fixed_returns_correct_token, test_mixed_inflation_per_request, test_prefill_only_request, test_various_spec_token_counts, ...`.
**CN:** 类 `TestBackupTokenAsyncSpec` 组织了 7 个测试方法。 代表性场景：`test_no_inflation_fixed_returns_last_token, test_inflation_buggy_returns_placeholder, test_inflation_fixed_returns_correct_token, test_mixed_inflation_per_request, test_prefill_only_request, test_various_spec_token_counts, ...`。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest, torch`.
- **CN:** 外部库：`numpy, pytest, torch`。
- **EN:** Standard-library support: `__future__`.
- **CN:** 标准库支持：`__future__`。
