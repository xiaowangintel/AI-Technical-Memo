# test_rope_rocm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rotary/test_rope_rocm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates rope rocm behavior in SGLang's rotary area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rotary 领域中与 rope rocm 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.layers.rotary_embedding import RotaryEmbedding
from sglang.srt.utils import get_bool_env_var, is_hip
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.layers.rotary_embedding`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.layers.rotary_embedding`, `sglang.srt.utils`。

### Lines 10-25: CI registration and metadata / CI 注册与元数据
```python
register_amd_ci(est_time=3, suite="stage-b-test-1-gpu-small-amd")

torch.manual_seed(0)

_is_hip = is_hip()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip


_CASES = [
    (64, 64, 32, 8000, True, torch.bfloat16, "cuda", 32, 32, 1, 1),
    (256, 128, 4096, 10000, True, torch.bfloat16, "cuda", 2, 512, 4, 2),
    (512, 128, 311, 10000, True, torch.bfloat16, "cuda", 3, 39, 4, 2),
    (128, 128, 2048, 10000, False, torch.bfloat16, "cuda", 2, 512, 32, 8),
    (128, 128, 2048, 10000, False, torch.bfloat16, "cuda", 2, 512, 16, 4),
    (512, 128, 311, 10000, False, torch.bfloat16, "cuda", 3, 39, 4, 2),
]
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_amd_ci, manual_seed, is_hip.
**CN:** 该代码块通过 register_amd_ci, manual_seed, is_hip 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 28-30: class TestRotaryEmbeddingNative declaration / 类 TestRotaryEmbeddingNative 声明
```python
@unittest.skipIf(_use_aiter, reason="SGLANG_USE_AITER=1 will not use vllm path.")
class TestRotaryEmbeddingNative(CustomTestCase):
    # Compare RotaryEmbedding.forward_hip() to forward_native().
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 31-64: method run case / 方法 run case
```python
    def _run_case(
        self,
        head_size: int,
        rotary_dim: int,
        max_pos: int,
        base: int,
        is_neox: bool,
        dtype: torch.dtype,
        device: str,
        batch_size: int,
        seq_len: int,
        num_q: int,
        num_kv: int,
    ) -> None:
        rope_ref = RotaryEmbedding(
            head_size, rotary_dim, max_pos, base, is_neox, dtype
        ).to(device)
        rope_hip = RotaryEmbedding(
            head_size, rotary_dim, max_pos, base, is_neox, dtype
        ).to(device)

        pos_ids = torch.arange(seq_len, device=device).repeat(batch_size)
        query = torch.randn(
            batch_size * seq_len, num_q * head_size, dtype=dtype, device=device
        )
        key = torch.randn(
            batch_size * seq_len, num_kv * head_size, dtype=dtype, device=device
        )

        q_ref, k_ref = rope_ref.forward_native(pos_ids, query.clone(), key.clone())
        q_hip, k_hip = rope_hip.forward_hip(pos_ids, query.clone(), key.clone())

        torch.testing.assert_close(q_ref, q_hip, atol=1e-2, rtol=1e-2)
        torch.testing.assert_close(k_ref, k_hip, atol=1e-2, rtol=1e-2)
```
**EN:** This block implements `_run_case` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_case`，承担模块行为中的一个聚焦逻辑片段。

### Lines 66-70: test case all cases / 测试用例 all cases
```python
    def test_all_cases(self) -> None:
        """Drive over the full parameter matrix using subTest()."""
        for case in _CASES:
            with self.subTest(case=case):
                self._run_case(*case)
```
**EN:** Drive over the full parameter matrix using subTest(). This test exercises `test_all_cases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Drive over the full parameter matrix using subTest(). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_cases`。

### Lines 73-78: class TestRotaryEmbeddingAITer declaration / 类 TestRotaryEmbeddingAITer 声明
```python
@unittest.skipIf(not _use_aiter, reason="Requires AMD GPU plus SGLANG_USE_AITER=1")
class TestRotaryEmbeddingAITer(CustomTestCase):
    # NOTE: Slightly relaxed tolerance (2e-2 vs 1e-2) for AITER RoPE kernel.
    # Minor precision differences under investigation.
    # See: https://github.com/sgl-project/sglang/pull/15318

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 79-114: method run case aiter / 方法 run case aiter
```python
    @staticmethod
    def _run_case_aiter(
        head_size: int,
        rotary_dim: int,
        max_pos: int,
        base: int,
        is_neox: bool,
        dtype: torch.dtype,
        device: str,
        batch_size: int,
        seq_len: int,
        num_q: int,
        num_kv: int,
    ) -> None:
        from aiter.rotary_embedding import RotaryEmbedding as AiterRotaryEmbedding

        rope_ref = AiterRotaryEmbedding(
            head_size, rotary_dim, max_pos, base, is_neox, dtype
        ).to(device)
        rope_hip = AiterRotaryEmbedding(
            head_size, rotary_dim, max_pos, base, is_neox, dtype
        ).to(device)

        pos_ids = torch.arange(seq_len, device=device).repeat(batch_size)
        query = torch.randn(
            batch_size * seq_len, num_q * head_size, dtype=dtype, device=device
        )
        key = torch.randn(
            batch_size * seq_len, num_kv * head_size, dtype=dtype, device=device
        )

        q_ref, k_ref = rope_ref.forward_native(pos_ids, query.clone(), key.clone())
        q_hip, k_hip = rope_hip.forward_hip(pos_ids, query.clone(), key.clone())

        torch.testing.assert_close(q_ref, q_hip, atol=2e-2, rtol=2e-2)
        torch.testing.assert_close(k_ref, k_hip, atol=2e-2, rtol=2e-2)
```
**EN:** This block implements `_run_case_aiter` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_case_aiter`，承担模块行为中的一个聚焦逻辑片段。

### Lines 116-119: test case all cases / 测试用例 all cases
```python
    def test_all_cases(self) -> None:
        for case in _CASES:
            with self.subTest(case=case):
                self._run_case_aiter(*case)
```
**EN:** This test exercises `test_all_cases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_cases`。

### Lines 122-123: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestRotaryEmbeddingNative`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRotaryEmbeddingAITer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRotaryEmbeddingNative._run_case`: This block implements `_run_case` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_case`，承担模块行为中的一个聚焦逻辑片段。
- `TestRotaryEmbeddingNative.test_all_cases`: Drive over the full parameter matrix using subTest(). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_cases`。
- `TestRotaryEmbeddingAITer._run_case_aiter`: This block implements `_run_case_aiter` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_case_aiter`，承担模块行为中的一个聚焦逻辑片段。
- `TestRotaryEmbeddingAITer.test_all_cases`: This test exercises `test_all_cases` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_cases`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers.rotary_embedding`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 123
