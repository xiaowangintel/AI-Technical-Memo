# test_spec_decode_acceptance.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/nixl_integration/test_spec_decode_acceptance.py`
- **Repository**: vllm-project/vllm
- **Purpose**: NixlConnector PD + EAGLE3 speculative decoding acceptance length test. / 该文件的文档字符串表明其用途：`nixlconnector pd + eagle3 speculative decoding acceptance length test`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-17)
```python
"""NixlConnector PD + EAGLE3 speculative decoding acceptance length test.

  - Loads MT-Bench prompts (80 prompts, 256 output tokens)
  - Sends through the PD proxy (completions API)
  - Scrapes Prometheus metrics from the decode server
  - Asserts acceptance length matches standalone EAGLE3 baselines

Baselines from tests/v1/spec_decode/test_acceptance_length.py
(standalone EAGLE3 with same model/drafter on MT-Bench, temp=0).
PD disaggregation via NixlConnector should match within tolerance.

Environment variables (set by spec_decode_acceptance_test.sh):
    TEST_MODEL   - target model name
    DECODE_PORT  - port of the decode vLLM server (for /metrics)
"""
```
**EN:** Module docstring that declares the scope of the file: NixlConnector PD + EAGLE3 speculative decoding acceptance length test.
**CN:** 模块文档字符串直接说明了文件范围：`nixlconnector pd + eagle3 speculative decoding acceptance length test`。

### Imports and setup / 导入与设置 (lines 19-28)
```python
import os
from dataclasses import dataclass, field
from types import SimpleNamespace
from urllib.request import urlopen

import openai
import regex as re
from transformers import AutoTokenizer

from vllm.benchmarks.datasets import get_samples
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `openai, regex, transformers`. vLLM modules under test include `vllm.benchmarks.datasets`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `openai, regex, transformers`。 被测试的 vLLM 模块包括 `vllm.benchmarks.datasets`。

### Module state / 模块级状态 (lines 30-33)
```python
SERVER_HOST = os.environ.get("SERVER_HOST", "127.0.0.1")
PROXY_BASE_URL = f"http://{SERVER_HOST}:8192/v1"
DECODE_PORT = os.environ.get("DECODE_PORT", "8200")
MODEL_NAME = os.environ.get("TEST_MODEL", "meta-llama/Llama-3.1-8B-Instruct")
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `SERVER_HOST, PROXY_BASE_URL, DECODE_PORT, MODEL_NAME`. Shared setup calls include `environ.get`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`SERVER_HOST, PROXY_BASE_URL, DECODE_PORT, MODEL_NAME`。 共享初始化调用包括 `environ.get`。

### Eagle3ModelConfig (lines 37-43)
```python
class Eagle3ModelConfig:
    verifier: str
    drafter: str
    expected_acceptance_length: float
    expected_acceptance_lengths_per_pos: list[float] = field(default_factory=list)
    id: str = ""
    rtol: float | None = None
```
**EN:** Class `Eagle3ModelConfig` groups 0 test method(s).
**CN:** 类 `Eagle3ModelConfig` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 48-60)
```python
EAGLE3_MODEL_CONFIGS = [
    Eagle3ModelConfig(
        verifier="meta-llama/Llama-3.1-8B-Instruct",
        drafter="RedHatAI/Llama-3.1-8B-Instruct-speculator.eagle3",
        expected_acceptance_length=2.60,
        expected_acceptance_lengths_per_pos=[0.7296, 0.5208, 0.3545],
        id="llama3-8b-eagle3",
    ),
]

DEFAULT_NUM_PROMPTS = 80
DEFAULT_OUTPUT_LEN = 256
DEFAULT_RTOL = 0.05
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `EAGLE3_MODEL_CONFIGS, DEFAULT_NUM_PROMPTS, DEFAULT_OUTPUT_LEN, DEFAULT_RTOL`. Shared setup calls include `Eagle3ModelConfig`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`EAGLE3_MODEL_CONFIGS, DEFAULT_NUM_PROMPTS, DEFAULT_OUTPUT_LEN, DEFAULT_RTOL`。 共享初始化调用包括 `Eagle3ModelConfig`。

### _get_model_config (lines 63-71)
```python
def _get_model_config() -> Eagle3ModelConfig:
    """Get the model config matching MODEL_NAME."""
    for config in EAGLE3_MODEL_CONFIGS:
        if config.verifier == MODEL_NAME:
            return config
    raise ValueError(
        f"No Eagle3ModelConfig found for model {MODEL_NAME}. "
        f"Available: {[c.verifier for c in EAGLE3_MODEL_CONFIGS]}"
    )
```
**EN:** Helper function `_get_model_config` encapsulates reusable logic for `model config`. Key calls include `ValueError`.
**CN:** 辅助函数 `_get_model_config` 封装了与 `model config` 相关的可复用逻辑。 关键调用包括 `ValueError`。

### _get_mt_bench_prompts (lines 74-100)
```python
def _get_mt_bench_prompts() -> list[str]:
    """Load MT-Bench prompts via vllm.benchmarks.datasets.get_samples."""
    tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)
    args = SimpleNamespace(
        dataset_name="hf",
        dataset_path="philschmid/mt-bench",
        num_prompts=DEFAULT_NUM_PROMPTS,
        seed=42,
        no_oversample=False,
        endpoint_type="openai-chat",
        backend="openai-chat",
        input_len=None,
        output_len=DEFAULT_OUTPUT_LEN,
        sharegpt_output_len=DEFAULT_OUTPUT_LEN,
        hf_name=None,
        hf_split="train",
        hf_subset=None,
        hf_output_len=DEFAULT_OUTPUT_LEN,
        no_stream=True,
        disable_shuffle=False,
        skip_chat_template=False,
        trust_remote_code=False,
        enable_multimodal_chat=False,
        request_id_prefix="",
    )
    samples = get_samples(args, tokenizer)
    return [sample.prompt for sample in samples]
```
**EN:** Helper function `_get_mt_bench_prompts` encapsulates reusable logic for `mt bench prompts`. Key calls include `AutoTokenizer.from_pretrained, SimpleNamespace, get_samples`.
**CN:** 辅助函数 `_get_mt_bench_prompts` 封装了与 `mt bench prompts` 相关的可复用逻辑。 关键调用包括 `AutoTokenizer.from_pretrained, SimpleNamespace, get_samples`。

### _fetch_metric (lines 103-110)
```python
def _fetch_metric(metric_name: str) -> float:
    """Fetch a single counter metric from the decode server's /metrics."""
    url = f"http://{SERVER_HOST}:{DECODE_PORT}/metrics"
    body = urlopen(url).read().decode()
    for line in body.split("\n"):
        if line.startswith(metric_name + "{") or line.startswith(metric_name + " "):
            return float(line.rsplit(" ", 1)[-1])
    raise ValueError(f"Metric {metric_name} not found in decode /metrics")
```
**EN:** Helper function `_fetch_metric` encapsulates reusable logic for `fetch metric`. Inputs: `metric_name`. Key calls include `read.decode, body.split, ValueError, urlopen.read, line.startswith, float`.
**CN:** 辅助函数 `_fetch_metric` 封装了与 `fetch metric` 相关的可复用逻辑。 输入参数：`metric_name`。 关键调用包括 `read.decode, body.split, ValueError, urlopen.read, line.startswith, float`。

### _fetch_per_position_acceptance (lines 113-126)
```python
def _fetch_per_position_acceptance() -> dict[int, float]:
    """Fetch per-position acceptance counts from decode /metrics."""
    url = f"http://{SERVER_HOST}:{DECODE_PORT}/metrics"
    body = urlopen(url).read().decode()
    counts: dict[int, float] = {}
    for line in body.split("\n"):
        if (
            "spec_decode_num_accepted_tokens_per_pos_total" in line
            and not line.startswith("#")
        ):
            m = re.search(r'position="(\d+)"', line)
            if m:
                counts[int(m.group(1))] = float(line.rsplit(" ", 1)[-1])
    return counts
```
**EN:** Helper function `_fetch_per_position_acceptance` encapsulates reusable logic for `fetch per position acceptance`. Key calls include `read.decode, body.split, urlopen.read, re.search, line.startswith, float`.
**CN:** 辅助函数 `_fetch_per_position_acceptance` 封装了与 `fetch per position acceptance` 相关的可复用逻辑。 关键调用包括 `read.decode, body.split, urlopen.read, re.search, line.startswith, float`。

### test_spec_decode_acceptance_length (lines 129-209)
```python
def test_spec_decode_acceptance_length():
    """Validate PD+SD acceptance length against standalone baseline.

    Sends MT-Bench prompts through the PD proxy (completions API),
    then checks that the decode server's speculative decoding metrics
    match the known standalone baselines.
    """
    config = _get_model_config()
    rtol = config.rtol if config.rtol is not None else DEFAULT_RTOL
    prompts = _get_mt_bench_prompts()
    assert len(prompts) == DEFAULT_NUM_PROMPTS, (
        f"Expected {DEFAULT_NUM_PROMPTS} prompts, got {len(prompts)}"
    )
    client = openai.OpenAI(api_key="EMPTY", base_url=PROXY_BASE_URL)
    for i, prompt in enumerate(prompts):
        resp = client.completions.create(
    # ... excerpt omitted for brevity ...
    assert n_drafts > 0, "No spec-decode drafts were generated"
        for i in range(len(config.expected_acceptance_lengths_per_pos))
    expected = config.expected_acceptance_length
    expected_per_pos = config.expected_acceptance_lengths_per_pos
        f"(expected={expected:.3f})"
    for i, (actual, exp) in enumerate(zip(per_pos_rates, expected_per_pos)):
            )
    print(
        f"\n=== PASS: {config.id} acceptance length {acceptance_length:.3f} "
        f"within {rtol:.0%} of {expected:.3f} ==="
```
**EN:** Test case covering `spec decode acceptance length`. It exercises `_get_model_config, _get_mt_bench_prompts, openai.OpenAI, enumerate, _fetch_metric, _fetch_per_position_acceptance`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `推测解码 acceptance length` 的测试用例。 该测试会调用 `_get_model_config, _get_mt_bench_prompts, openai.OpenAI, enumerate, _fetch_metric, _fetch_per_position_acceptance`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `openai, regex, transformers`.
- **CN:** 外部库：`openai, regex, transformers`。
- **EN:** vLLM modules under test: `vllm.benchmarks.datasets`.
- **CN:** 被测试的 vLLM 模块：`vllm.benchmarks.datasets`。
- **EN:** Standard-library support: `os, dataclasses, types, urllib.request`.
- **CN:** 标准库支持：`os, dataclasses, types, urllib.request`。
