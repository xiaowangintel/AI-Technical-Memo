# test_ascend_vocab_mask.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/ascend/test_ascend_vocab_mask.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `ascend vocab mask` scenario in `test/manual/ascend`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/ascend` 中的 `ascend vocab mask` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and shared helpers / 导入与共享辅助项
```python
import math

import pytest
import torch

from sglang.srt.constrained import xgrammar_backend as xb
```
**EN:** This range imports `math`, `pytest`, `torch` and `sglang.srt.constrained`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 9-15: Helper routines around _pack_mask / 辅助例程
```python
def _pack_mask(allowed_ids, vocab_size, batch_size=1):
    nwords = math.ceil(vocab_size / 32)
    m = torch.zeros((batch_size, nwords), dtype=torch.int32)
    for b in range(batch_size):
        for tid in allowed_ids[b]:
            m[b, tid // 32] |= 1 << (tid % 32)
    return m
```
**EN:** This range implements helper routine(s) `_pack_mask` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `ceil` and `zeros`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 18-27: Helper routines around _apply_ref_cpu / 辅助例程
```python
def _apply_ref_cpu(logits, vocab_mask):
    vocab_size = logits.shape[-1]
    token_ids = torch.arange(vocab_size, device="cpu", dtype=torch.int64)
    word_idx = token_ids // 32
    bit_idx = (token_ids % 32).to(torch.int32)
    words = vocab_mask.cpu()[:, word_idx].to(torch.int32)
    allowed = ((words >> bit_idx) & 1).bool().to(logits.device)
    out = logits.clone()
    out.masked_fill_(~allowed, float("-inf"))
    return out
```
**EN:** This range implements helper routine(s) `_apply_ref_cpu` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `arange`, `to`, `cpu` and `bool`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 28-32: Scenario logic / 场景逻辑
```python


@pytest.mark.skipif(
    not hasattr(torch, "npu") or not torch.npu.is_available(), reason="NPU required"
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipif`, `hasattr` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 33-49: Test routines around test_mask_blocks_disallowed_token_on_npu / 测试例程
```python
def test_mask_blocks_disallowed_token_on_npu():
    device = "npu:0"
    vocab_size = 64

    logits = torch.zeros((1, vocab_size), device=device, dtype=torch.float32)
    logits[0, 16] = 22.125
    logits[0, 5] = 10.0

    allowed = [[5, 6, 7, 8]]
    vocab_mask = _pack_mask(allowed, vocab_size).to(device=device, dtype=torch.int32)

    g = xb.XGrammarGrammar.__new__(xb.XGrammarGrammar)
    out = logits.clone()
    g.apply_vocab_mask(out, vocab_mask)

    assert not torch.isfinite(out[0, 16])
    assert int(torch.argmax(out[0]).item()) != 16
```
**EN:** This range defines concrete test routine(s) `test_mask_blocks_disallowed_token_on_npu`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `zeros`, `_pack_mask`, `to` and `__new__`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-54: Scenario logic / 场景逻辑
```python


@pytest.mark.skipif(
    not hasattr(torch, "npu") or not torch.npu.is_available(), reason="NPU required"
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipif`, `hasattr` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 55-79: Test routines around test_npu_path_matches_reference_random / 测试例程
```python
def test_npu_path_matches_reference_random():
    device = "npu:0"
    B, V = 4, 257
    torch.manual_seed(0)

    logits = torch.randn(B, V, device=device, dtype=torch.float32)

    allowed = []
    for _ in range(B):
        ids = torch.randperm(V)[: V // 4].tolist()
        allowed.append(ids)
    vocab_mask = _pack_mask(allowed, V, B).to(device=device, dtype=torch.int32)

    g = xb.XGrammarGrammar.__new__(xb.XGrammarGrammar)
    out_npu = logits.clone()
    g.apply_vocab_mask(out_npu, vocab_mask)

    out_ref = _apply_ref_cpu(logits, vocab_mask)

    assert torch.equal(torch.isfinite(out_npu), torch.isfinite(out_ref))
    diff = (
        torch.nan_to_num(out_npu - out_ref, nan=0.0, posinf=0.0, neginf=0.0)
        .abs()
        .max()
        .item()
```
**EN:** This range defines concrete test routine(s) `test_npu_path_matches_reference_random`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `manual_seed`, `randn`, `randperm` and `tolist`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 80-81: Assertions and result checks / 断言与结果检查
```python
    )
    assert diff < 1e-5
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。

## Key Concepts / 关键概念
- Pytest markers / Pytest 标记
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `math`
- **Third-party / 第三方库**: `pytest`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.constrained`
