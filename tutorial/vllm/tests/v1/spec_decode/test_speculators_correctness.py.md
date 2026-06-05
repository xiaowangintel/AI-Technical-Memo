# test_speculators_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/spec_decode/test_speculators_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `speculators correctness` behavior and regressions in the v1 stack. / 验证 v1 栈中 `speculators correctness` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-12)
```python
import dataclasses

import pytest
import torch

from tests.evals.gsm8k.gsm8k_eval import evaluate_gsm8k_offline
from tests.utils import large_gpu_mark
from vllm import LLM
from vllm.config import SpeculativeConfig
from vllm.distributed import cleanup_dist_env_and_memory
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm, vllm.config, vllm.distributed`. Local helpers come from `tests.evals.gsm8k.gsm8k_eval, tests.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm, vllm.config, vllm.distributed`。 本地测试辅助逻辑来自 `tests.evals.gsm8k.gsm8k_eval, tests.utils`。

### SpeculatorTestConfig (lines 16-27)
```python
class SpeculatorTestConfig:
    model_path: str
    method: str
    display_name: str
    expected_gsm8k_accuracy: float
    accuracy_rtol: float
    expected_acceptance_len: float
    acceptance_len_rtol: float
    expected_per_pos_acceptance_rates: tuple[float, ...]
    per_pos_rtol: float
    quantization: str | None = None
    parallel_drafting: bool | None = None
```
**EN:** Class `SpeculatorTestConfig` groups 0 test method(s).
**CN:** 类 `SpeculatorTestConfig` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 30-59)
```python
DFLASH_CONFIG = SpeculatorTestConfig(
    model_path="nm-testing/dflash-qwen3-8b-speculators",
    method="dflash",
    display_name="DFlash",
    expected_gsm8k_accuracy=0.885,
    accuracy_rtol=0.03,
    expected_acceptance_len=3.45,
    acceptance_len_rtol=0.15,
    expected_per_pos_acceptance_rates=(0.795, 0.611, 0.429, 0.282),
    per_pos_rtol=0.15,
    quantization="fp8",
)

PEAGLE_CONFIG = SpeculatorTestConfig(
    model_path="nm-testing/qwen3-8b-peagle-speculators",
    method="eagle3",
    display_name="PEagle",
    expected_gsm8k_accuracy=0.88,
    accuracy_rtol=0.05,
    expected_acceptance_len=2.27,
    acceptance_len_rtol=0.20,
    expected_per_pos_acceptance_rates=(0.66, 0.36, 0.18, 0.09),
    per_pos_rtol=0.20,
    parallel_drafting=True,
)

SPECULATOR_CONFIGS = [
    pytest.param(DFLASH_CONFIG, id="dflash"),
    pytest.param(PEAGLE_CONFIG, id="peagle"),
]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DFLASH_CONFIG, PEAGLE_CONFIG, SPECULATOR_CONFIGS`. Shared setup calls include `SpeculatorTestConfig, pytest.param`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DFLASH_CONFIG, PEAGLE_CONFIG, SPECULATOR_CONFIGS`。 共享初始化调用包括 `SpeculatorTestConfig, pytest.param`。

### compute_spec_decode_stats (lines 62-86)
```python
def compute_spec_decode_stats(metrics) -> dict:
    """Extract all spec-decode metrics and compute derived stats."""
    name2metric = {m.name: m for m in metrics}

    n_drafts = name2metric["vllm:spec_decode_num_drafts"].value
    n_draft_tokens = name2metric["vllm:spec_decode_num_draft_tokens"].value
    n_accepted = name2metric["vllm:spec_decode_num_accepted_tokens"].value

    per_pos_vec = name2metric["vllm:spec_decode_num_accepted_tokens_per_pos"].values

    acceptance_len = 1 + (n_accepted / n_drafts) if n_drafts > 0 else 1.0
    draft_tokens_per_step = (n_draft_tokens / n_drafts) if n_drafts > 0 else 0
    overall_acceptance_rate = (n_accepted / n_draft_tokens) if n_draft_tokens > 0 else 0
    per_pos_rates = [v / n_drafts for v in per_pos_vec] if n_drafts > 0 else []

    return {
        "num_drafts": n_drafts,
        "num_draft_tokens": n_draft_tokens,
        "num_accepted_tokens": n_accepted,
        "acceptance_len": acceptance_len,
        "draft_tokens_per_step": draft_tokens_per_step,
        "overall_acceptance_rate": overall_acceptance_rate,
        "per_pos_accepted": list(per_pos_vec),
        "per_pos_acceptance_rates": per_pos_rates,
    }
```
**EN:** Helper function `compute_spec_decode_stats` encapsulates reusable logic for `compute spec decode stats`. Inputs: `metrics`. Key calls include `list`.
**CN:** 辅助函数 `compute_spec_decode_stats` 封装了与 `compute 推测解码 stats` 相关的可复用逻辑。 输入参数：`metrics`。 关键调用包括 `list`。

### print_spec_decode_stats (lines 89-102)
```python
def print_spec_decode_stats(stats: dict) -> None:
    """Print all spec-decode metrics and derived values."""
    print("\n===== Spec Decode Metrics =====")
    print(f"  num_drafts:              {stats['num_drafts']}")
    print(f"  num_draft_tokens:        {stats['num_draft_tokens']}")
    print(f"  num_accepted_tokens:     {stats['num_accepted_tokens']}")
    print(f"  draft_tokens_per_step:   {stats['draft_tokens_per_step']:.2f}")
    print(f"  overall_acceptance_rate: {stats['overall_acceptance_rate']:.4f}")
    print(f"  acceptance_len (1+acc/drafts): {stats['acceptance_len']:.4f}")
    print("  per-position accepted tokens:", stats["per_pos_accepted"])
    print("  per-position acceptance rates:")
    for i, rate in enumerate(stats["per_pos_acceptance_rates"]):
        print(f"    pos {i}: {rate:.4f}")
    print("===============================\n")
```
**EN:** Helper function `print_spec_decode_stats` encapsulates reusable logic for `print spec decode stats`. Inputs: `stats`. Key calls include `print, enumerate`.
**CN:** 辅助函数 `print_spec_decode_stats` 封装了与 `print 推测解码 stats` 相关的可复用逻辑。 输入参数：`stats`。 关键调用包括 `print, enumerate`。

### test_speculators_model (lines 106-151)
```python
def test_speculators_model(vllm_runner, example_prompts, monkeypatch, config):
    """
    Test speculators model properly initializes speculative decoding.

    Verifies:
    1. Speculative config is automatically initialized from speculators config
    2. Method is detected correctly
    3. parallel_drafting is set correctly (if applicable)
    4. The draft model path is correctly set
    5. Speculative tokens count is valid
    6. Text generation works with speculative decoding enabled
    monkeypatch.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
    runner_kwargs = dict(dtype=torch.bfloat16, enforce_eager=True)
    if config.quantization:
        runner_kwargs["quantization"] = config.quantization
    # ... excerpt omitted for brevity ...
        assert isinstance(vllm_config.speculative_config, SpeculativeConfig), (
        assert spec_config.method == config.method, (
            assert spec_config.parallel_drafting is config.parallel_drafting, (
        assert spec_config.num_speculative_tokens > 0, (
        assert spec_config.model == config.model_path, (
        )
        vllm_outputs = vllm_model.generate_greedy(example_prompts, max_tokens=20)
        assert vllm_outputs, (
            f"No outputs generated for speculators model {config.model_path}"
```
**EN:** Parameterized test covering `speculators model`. Parameter axes: `config`. Inputs/fixtures: `vllm_runner, example_prompts, monkeypatch, config`. It exercises `mark.parametrize, monkeypatch.setenv, dict, vllm_runner, isinstance, vllm_model.generate_greedy`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `speculators model` 的测试用例。 参数维度：`config`。 输入或 fixture：`vllm_runner, example_prompts, monkeypatch, config`。 该测试会调用 `mark.parametrize, monkeypatch.setenv, dict, vllm_runner, isinstance, vllm_model.generate_greedy`。 代码主体包含 6 个显式断言。

### test_speculators_correctness (lines 157-209)
```python
def test_speculators_correctness(monkeypatch, config):
    """
    E2E correctness test via the speculators auto-detect path.

    Evaluates GSM8k accuracy to ensure the speculators-format model produces
    correct outputs, and checks that acceptance length does not collapse under
    batched inference (lm-eval style).
    monkeypatch.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
    spec_llm = LLM(
        model=config.model_path,
        trust_remote_code=True,
        max_model_len=4096,
        max_num_seqs=128,
        gpu_memory_utilization=0.85,
        enforce_eager=False,
        disable_log_stats=False,
    # ... excerpt omitted for brevity ...
    accuracy_threshold = config.expected_gsm8k_accuracy * (1 - config.accuracy_rtol)
    assert accuracy >= accuracy_threshold, (
    al_threshold = config.expected_acceptance_len * (1 - config.acceptance_len_rtol)
    assert acceptance_len >= al_threshold, (
    for i, expected_rate in enumerate(config.expected_per_pos_acceptance_rates):
        assert i < len(per_pos_rates), (
            f"(expected ~{expected_rate:.4f})"
        )
    del spec_llm
    torch.accelerator.empty_cache()
    cleanup_dist_env_and_memory()
```
**EN:** Parameterized test covering `speculators correctness`. Parameter axes: `config`. Inputs/fixtures: `monkeypatch, config`. It exercises `large_gpu_mark, mark.parametrize, monkeypatch.setenv, LLM, evaluate_gsm8k_offline, spec_llm.get_metrics`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `speculators correctness` 的测试用例。 参数维度：`config`。 输入或 fixture：`monkeypatch, config`。 该测试会调用 `large_gpu_mark, mark.parametrize, monkeypatch.setenv, LLM, evaluate_gsm8k_offline, spec_llm.get_metrics`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm, vllm.config, vllm.distributed`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config, vllm.distributed`。
- **EN:** Local test helpers: `tests.evals.gsm8k.gsm8k_eval, tests.utils`.
- **CN:** 本地测试辅助模块：`tests.evals.gsm8k.gsm8k_eval, tests.utils`。
- **EN:** Standard-library support: `dataclasses`.
- **CN:** 标准库支持：`dataclasses`。
