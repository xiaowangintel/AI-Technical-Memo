# test_custom_dataset_seed.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/benchmarks/test_custom_dataset_seed.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Custom Dataset Seed behavior in the Benchmarks test area through focused pytest scenarios. It focuses on scenarios such as HF Tokenizer, Write Jsonl, Args For Custom. / 该文件在 Benchmarks 测试域中，通过有针对性的 pytest 场景验证 Custom Dataset Seed 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-10)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import argparse
import json
from pathlib import Path

import pytest
from transformers import AutoTokenizer, PreTrainedTokenizerBase

from vllm.benchmarks.datasets import get_samples
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `argparse`, `json`, `pytest`, `transformers`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: hf_tokenizer (lines 13-15)
```python
@pytest.fixture(scope="session")
def hf_tokenizer() -> PreTrainedTokenizerBase:
    return AutoTokenizer.from_pretrained("gpt2")
```
**EN:** Provides a pytest fixture for HF Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `AutoTokenizer.from_pretrained`.
**CN:** 该代码块定义 pytest 夹具 `hf_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `AutoTokenizer.from_pretrained` 构造或返回测试所需的值。

### Helper: _write_jsonl (lines 18-21)
```python
def _write_jsonl(path: Path, n_rows: int) -> None:
    with path.open("w") as f:
        for i in range(n_rows):
            f.write(json.dumps({"prompt": f"row {i}: unique prompt content."}) + "\n")
```
**EN:** Implements a reusable helper for Write Jsonl, reducing duplication across related tests. It coordinates operations such as `path.open`, `range`, `f.write`.
**CN:** 该辅助函数为 Write Jsonl 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `path.open`, `range`, `f.write` 等操作。

### Helper: _args_for_custom (lines 24-35)
```python
def _args_for_custom(dataset_path: str, seed: int) -> argparse.Namespace:
    return argparse.Namespace(
        dataset_name="custom",
        dataset_path=dataset_path,
        disable_shuffle=False,
        num_prompts=30,
        custom_output_len=32,
        skip_chat_template=True,
        no_oversample=False,
        seed=seed,
        request_id_prefix="",
    )
```
**EN:** Implements a reusable helper for Args For Custom, reducing duplication across related tests. It coordinates operations such as `argparse.Namespace`.
**CN:** 该辅助函数为 Args For Custom 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `argparse.Namespace` 等操作。

### Test: test_custom_dataset_seed_propagates (lines 38-60)
```python
@pytest.mark.benchmark
def test_custom_dataset_seed_propagates(
    hf_tokenizer: PreTrainedTokenizerBase, tmp_path: Path
) -> None:
    """--seed must control the CustomDataset shuffle used by get_samples.

    Without the fix, CustomDataset was instantiated without random_seed,
    so its load-time shuffle always used DEFAULT_SEED=0 regardless of
    args.seed, causing every run with --dataset-name custom to pick the
    same subset of rows from a larger file.
    """
    jsonl = tmp_path / "data.jsonl"
    _write_jsonl(jsonl, n_rows=60)

    samples_a = get_samples(_args_for_custom(str(jsonl), seed=0), hf_tokenizer)
    samples_b = get_samples(_args_for_custom(str(jsonl), seed=42), hf_tokenizer)

    prompts_a = {s.prompt for s in samples_a}
    prompts_b = {s.prompt for s in samples_b}

    assert len(prompts_a) == 30
    assert len(prompts_b) == 30
    assert prompts_a != prompts_b
```
**EN:** --seed must control the CustomDataset shuffle used by get_samples. The body exercises logic via `_write_jsonl`, `get_samples`, `_args_for_custom` before asserting the expected outcome.
**CN:** 该测试用例验证 Custom Dataset Seed Propagates 在特定场景下的行为。 函数体会先通过 `_write_jsonl`, `get_samples`, `_args_for_custom` 驱动目标逻辑，再断言预期结果。

### Test: test_custom_dataset_same_seed_is_deterministic (lines 63-77)
```python
@pytest.mark.benchmark
def test_custom_dataset_same_seed_is_deterministic(
    hf_tokenizer: PreTrainedTokenizerBase, tmp_path: Path
) -> None:
    """Same --seed must yield the same CustomDataset subset."""
    jsonl = tmp_path / "data.jsonl"
    _write_jsonl(jsonl, n_rows=60)

    samples_a = get_samples(_args_for_custom(str(jsonl), seed=7), hf_tokenizer)
    samples_b = get_samples(_args_for_custom(str(jsonl), seed=7), hf_tokenizer)

    prompts_a = [s.prompt for s in samples_a]
    prompts_b = [s.prompt for s in samples_b]

    assert prompts_a == prompts_b
```
**EN:** Same --seed must yield the same CustomDataset subset. The body exercises logic via `_write_jsonl`, `get_samples`, `_args_for_custom` before asserting the expected outcome.
**CN:** 该测试用例验证 Custom Dataset Same Seed Is Deterministic 在特定场景下的行为。 函数体会先通过 `_write_jsonl`, `get_samples`, `_args_for_custom` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Benchmarking / 基准测试**
  - **EN:** The tests validate CLI arguments, datasets, or metrics used by benchmark workflows.
  - **CN:** 测试验证基准流程使用的命令行参数、数据集或指标。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `pathlib`
- **Third-party / 第三方依赖**: `pytest`, `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm.benchmarks.datasets`
