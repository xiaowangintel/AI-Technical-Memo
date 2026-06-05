# test_rejection_sampler_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/spec_decode/test_rejection_sampler_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `rejection sampler utils` behavior and regressions in the v1 stack. / 验证 v1 栈中 `rejection sampler utils` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-11)
```python
import math

import pytest
import torch

from vllm.v1.worker.gpu.spec_decode.rejection_sampler_utils import (
    rejection_sample,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.v1.worker.gpu.spec_decode.rejection_sampler_utils, vllm.v1.spec_decode.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.v1.worker.gpu.spec_decode.rejection_sampler_utils, vllm.v1.spec_decode.utils`。

### Module state / 模块级状态 (lines 13-18)
```python
VOCAB_SIZE = 4096

# Skip if no CUDA - Triton kernel requires GPU
pytest.importorskip("triton")
if not torch.cuda.is_available():
    pytest.skip("CUDA required for rejection sampler tests", allow_module_level=True)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `VOCAB_SIZE`. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `pytest.importorskip, cuda.is_available, pytest.skip`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`VOCAB_SIZE`。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `pytest.importorskip, cuda.is_available, pytest.skip`。

### _build_rejection_sample_inputs (lines 21-73)
```python
def _build_rejection_sample_inputs(
    target_logits_1d: torch.Tensor,
    draft_logits_1d: torch.Tensor,
    num_speculative_steps: int,
    temperature: float,
    num_trials: int,
) -> dict:
    device = target_logits_1d.device
    vocab_size = target_logits_1d.shape[0]
    K = num_speculative_steps
    num_logits = num_trials * (K + 1)

    target_logits = target_logits_1d.unsqueeze(0).expand(num_logits, -1).contiguous()
    draft_logits = (
        draft_logits_1d.view(1, 1, vocab_size).expand(num_trials, K, -1).contiguous()
    )
    draft_probs = torch.softmax(draft_logits_1d, dim=0)
    # ... excerpt omitted for brevity ...
    return dict(
        idx_mapping=idx_mapping,
        expanded_idx_mapping=expanded_idx_mapping,
        expanded_local_pos=expanded_local_pos,
        temperature=temp_tensor,
        seed=seed,
```
**EN:** Helper function `_build_rejection_sample_inputs` encapsulates reusable logic for `rejection sample inputs`. Inputs: `target_logits_1d, draft_logits_1d, num_speculative_steps, temperature, num_trials`. Key calls include `expand.contiguous, torch.softmax, torch.multinomial, torch.zeros, draft_sampled_2d.reshape, torch.arange`.
**CN:** 辅助函数 `_build_rejection_sample_inputs` 封装了与 `rejection sample inputs` 相关的可复用逻辑。 输入参数：`target_logits_1d, draft_logits_1d, num_speculative_steps, temperature, num_trials`。 关键调用包括 `expand.contiguous, torch.softmax, torch.multinomial, torch.zeros, draft_sampled_2d.reshape, torch.arange`。

### _assert_distribution_match (lines 76-129)
```python
def _assert_distribution_match(
    sampled_tokens: torch.Tensor,
    target_probs: torch.Tensor,
    device: str,
    label: str = "",
    min_expected: float = 5.0,
):
    """
    Assert sampled tokens match the target distribution via a
    chi-squared goodness-of-fit test. This is done by computing
    observed vs expected token counts (target_probs * num_samples),
    then checking that the chi-squared statistic is below a conservative
    threshold. The threshold is set at df + 10*sqrt(2*df), which
    corresponds to ~10 sigma under the chi-squared distribution's
    normal approximation, effectively disallowing false positives.

    NOTE: Tokens with expected count < min_expected are merged into
    a single "other" bin to minimize chi-squared noise.
    # ... excerpt omitted for brevity ...
    expected = target_probs * num_samples
    sufficient = expected >= min_expected
    exp_main = expected[sufficient]
    exp_other = expected[~sufficient].sum().unsqueeze(0)
    if exp_other.item() >= min_expected:
    prefix = f"[{label}] " if label else ""
    assert chi2 < threshold, (
        f"{prefix}Chi-squared test failed: chi2={chi2:.1f}, "
        f"df={df}, threshold={threshold:.1f}. "
        f"Output distribution does not match target distribution."
    )
```
**EN:** Helper function `_assert_distribution_match` encapsulates reusable logic for `assert distribution match`. Inputs: `sampled_tokens, target_probs, device, label, min_expected`. Key calls include `torch.zeros, observed.scatter_add_, sum.unsqueeze, sum.item, torch.ones, exp_other.item`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_assert_distribution_match` 封装了与 `assert distribution match` 相关的可复用逻辑。 输入参数：`sampled_tokens, target_probs, device, label, min_expected`。 关键调用包括 `torch.zeros, observed.scatter_add_, sum.unsqueeze, sum.item, torch.ones, exp_other.item`。 其中包含 1 个内部断言，用于保护前置假设。

### test_stochastic_rejection_sample (lines 141-179)
```python
def test_stochastic_rejection_sample(num_speculative_steps: int, temperature: float):
    """
    Verify that rejection sampling produces the target distribution.
    This is done by simulating many independent trials of speculative
    decoding (from a fixed target and draft distribution). We then
    run rejection sample on all of the trials (requests), and verify
    that the sampled tokens at every position follow the target
    distribution p(x).

    torch.manual_seed(42)
    device = "cuda"
    num_trials = 10 * VOCAB_SIZE
    target_logits_1d = torch.randn(VOCAB_SIZE, device=device, dtype=torch.float32)
    draft_logits_1d = torch.randn(VOCAB_SIZE, device=device, dtype=torch.float32)
    if temperature > 0:
    # ... excerpt omitted for brevity ...
    target_probs = torch.softmax(target_logits_1d, dim=0)
    for pos in range(num_speculative_steps + 1):
        accepted_mask = num_sampled >= pos + 1
        _assert_distribution_match(
            sampled[accepted_mask, pos], target_probs, device, label=f"position {pos}"
        )
```
**EN:** Parameterized test covering `stochastic rejection sample`. Parameter axes: `num_speculative_steps, temperature`. Inputs/fixtures: `num_speculative_steps, temperature`. It exercises `mark.parametrize, torch.manual_seed, torch.randn, _build_rejection_sample_inputs, rejection_sample, torch.softmax`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `stochastic rejection sample` 的测试用例。 参数维度：`num_speculative_steps, temperature`。 输入或 fixture：`num_speculative_steps, temperature`。 该测试会调用 `mark.parametrize, torch.manual_seed, torch.randn, _build_rejection_sample_inputs, rejection_sample, torch.softmax`。 主要通过 mock、回调或输出检查来完成验证。

### test_greedy_rejection_sample (lines 183-215)
```python
def test_greedy_rejection_sample(num_speculative_steps: int):
    """
    Verify that greedy (temperature=0) always outputs the target argmax
    at every accepted position.

    torch.manual_seed(42)
    device = "cuda"
    num_trials = 10 * VOCAB_SIZE
    target_logits_1d = torch.randn(VOCAB_SIZE, device=device, dtype=torch.float32)
    draft_logits_1d = torch.randn(VOCAB_SIZE, device=device, dtype=torch.float32)
    inputs = _build_rejection_sample_inputs(
        target_logits_1d,
        draft_logits_1d,
        num_speculative_steps,
        temperature=0.0,
    # ... excerpt omitted for brevity ...
    steps = torch.arange(num_speculative_steps + 1, device=device).unsqueeze(0)
    accepted_mask = steps < num_sampled.unsqueeze(1)
    assert (sampled[accepted_mask] == target_argmax).all(), (
        "Greedy sampling produced tokens that are not the target argmax"
    )
```
**EN:** Parameterized test covering `greedy rejection sample`. Parameter axes: `num_speculative_steps`. Inputs/fixtures: `num_speculative_steps`. It exercises `mark.parametrize, torch.manual_seed, torch.randn, _build_rejection_sample_inputs, rejection_sample, argmax.item`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `greedy rejection sample` 的测试用例。 参数维度：`num_speculative_steps`。 输入或 fixture：`num_speculative_steps`。 该测试会调用 `mark.parametrize, torch.manual_seed, torch.randn, _build_rejection_sample_inputs, rejection_sample, argmax.item`。 代码主体包含 1 个显式断言。

### test_synthetic_rejection_sample (lines 231-282)
```python
def test_synthetic_rejection_sample(
    num_speculative_steps: int,
    temperature: float,
    unconditional_rates: list[float],
):
    """
    Verify that synthetic rejection sampling produces the expected
    per-position acceptance rates. The unconditional rate at position i
    is P(all draft steps 0..i accepted) = product(conditional_rates[0:i+1]).
    This is approximately mean(num accepted >= i + 1) over many trials.
    from vllm.v1.spec_decode.utils import unconditional_to_conditional_rates

    torch.manual_seed(42)
    device = "cuda"
    num_trials = 10 * VOCAB_SIZE
    deviation_tol = 1e-2
    # ... excerpt omitted for brevity ...
    for i, expected_rate in enumerate(unconditional_rates):
        observed_rate = (num_accepted >= i + 1).float().mean().item()
        assert abs(observed_rate - expected_rate) < deviation_tol, (
            f"Step {i}: observed rate {observed_rate:.4f} deviates from "
            f"expected rate {expected_rate:.4f} by more than {deviation_tol}."
        )
```
**EN:** Parameterized test covering `synthetic rejection sample`. Parameter axes: `num_speculative_steps, temperature, unconditional_rates`. Inputs/fixtures: `num_speculative_steps, temperature, unconditional_rates`. It exercises `mark.parametrize, torch.manual_seed, torch.randn, _build_rejection_sample_inputs, unconditional_to_conditional_rates, torch.tensor`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `synthetic rejection sample` 的测试用例。 参数维度：`num_speculative_steps, temperature, unconditional_rates`。 输入或 fixture：`num_speculative_steps, temperature, unconditional_rates`。 该测试会调用 `mark.parametrize, torch.manual_seed, torch.randn, _build_rejection_sample_inputs, unconditional_to_conditional_rates, torch.tensor`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.v1.worker.gpu.spec_decode.rejection_sampler_utils, vllm.v1.spec_decode.utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.worker.gpu.spec_decode.rejection_sampler_utils, vllm.v1.spec_decode.utils`。
- **EN:** Standard-library support: `math`.
- **CN:** 标准库支持：`math`。
