# test_multi_connector_edge_cases.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/nixl_integration/test_multi_connector_edge_cases.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Integration edge-case tests for MultiConnector (NixlConnector + OffloadingConnector). / 该文件的文档字符串表明其用途：`integration edge-case tests for multiconnector (nixlconnector + offloadingconnector)`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-13)
```python
"""
Integration edge-case tests for MultiConnector (NixlConnector + OffloadingConnector).

Tests cover:
  - Output correctness across block-size boundaries (proxy vs direct prefill).
  - Decode-side Prometheus metrics validation (local_cache_hit,
    external_kv_transfer, local_compute) for cold/warm/partial cache scenarios.
  - Prefill-side CPU offload recovery after GPU cache eviction.

Requires running servers started by run_multi_connector_edge_case_test.sh.
"""
```
**EN:** Module docstring that declares the scope of the file: Integration edge-case tests for MultiConnector (NixlConnector + OffloadingConnector).
**CN:** 模块文档字符串直接说明了文件范围：`integration edge-case tests for multiconnector (nixlconnector + offloadingconnector)`。

### Imports and setup / 导入与设置 (lines 15-20)
```python
import os
import time
import urllib.request

import openai
import regex as re
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `openai, regex`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `openai, regex`。

### Module state / 模块级状态 (lines 24-47)
```python
PREFILL_HOST = os.getenv("PREFILL_HOST", "localhost")
PREFILL_PORT = os.environ["PREFILL_PORT"]
DECODE_HOST = os.getenv("DECODE_HOST", "localhost")
DECODE_PORT = os.environ["DECODE_PORT"]
PROXY_HOST = os.getenv("PROXY_HOST", "localhost")
PROXY_PORT = os.environ["PROXY_PORT"]
BLOCK_SIZE = int(os.getenv("BLOCK_SIZE", "128"))

# ── OpenAI clients ────────────────────────────────────────────────────────

decode_client = openai.OpenAI(
    api_key="EMPTY",
    base_url=f"http://{DECODE_HOST}:{DECODE_PORT}/v1",
)
prefill_client = openai.OpenAI(
    api_key="EMPTY",
    base_url=f"http://{PREFILL_HOST}:{PREFILL_PORT}/v1",
)
proxy_client = openai.OpenAI(
    api_key="EMPTY",
    base_url=f"http://{PROXY_HOST}:{PROXY_PORT}/v1",
)

_MODEL = None
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `PREFILL_HOST, PREFILL_PORT, DECODE_HOST, DECODE_PORT, PROXY_HOST, PROXY_PORT, ...`. Shared setup calls include `os.getenv, int, openai.OpenAI`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`PREFILL_HOST, PREFILL_PORT, DECODE_HOST, DECODE_PORT, PROXY_HOST, PROXY_PORT, ...`。 共享初始化调用包括 `os.getenv, int, openai.OpenAI`。

### _get_model (lines 50-55)
```python
def _get_model() -> str:
    global _MODEL
    if _MODEL is None:
        models = decode_client.models.list()
        _MODEL = models.data[0].id
    return _MODEL
```
**EN:** Helper function `_get_model` encapsulates reusable logic for `model`. Key calls include `models.list`.
**CN:** 辅助函数 `_get_model` 封装了与 `model` 相关的可复用逻辑。 关键调用包括 `models.list`。

### _complete (lines 58-66)
```python
def _complete(client: openai.OpenAI, prompt: str, max_tokens: int = 20):
    """Send a completion request and return (text, prompt_tokens)."""
    resp = client.completions.create(
        model=_get_model(),
        prompt=prompt,
        max_tokens=max_tokens,
        temperature=0,
    )
    return resp.choices[0].text, resp.usage.prompt_tokens
```
**EN:** Helper function `_complete` encapsulates reusable logic for `complete`. Inputs: `client, prompt, max_tokens`. Key calls include `completions.create, _get_model`.
**CN:** 辅助函数 `_complete` 封装了与 `complete` 相关的可复用逻辑。 输入参数：`client, prompt, max_tokens`。 关键调用包括 `completions.create, _get_model`。

### Module state / 模块级状态 (lines 71-74)
```python
_METRIC_RE = re.compile(
    r'vllm:prompt_tokens_by_source_total\{.*?source="([^"]+)".*?\}\s+'
    r"([\d.eE+\-]+)"
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_METRIC_RE`. Shared setup calls include `re.compile`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_METRIC_RE`。 共享初始化调用包括 `re.compile`。

### _fetch_metrics (lines 77-89)
```python
def _fetch_metrics(host: str, port: str) -> dict[str, float]:
    """Scrape prompt_tokens_by_source counters from a vLLM server."""
    body = urllib.request.urlopen(f"http://{host}:{port}/metrics").read().decode()
    result = {
        "local_compute": 0.0,
        "local_cache_hit": 0.0,
        "external_kv_transfer": 0.0,
    }
    for m in _METRIC_RE.finditer(body):
        source, val = m.group(1), float(m.group(2))
        if source in result:
            result[source] += val
    return result
```
**EN:** Helper function `_fetch_metrics` encapsulates reusable logic for `fetch metrics`. Inputs: `host, port`. Key calls include `read.decode, _METRIC_RE.finditer, urlopen.read, m.group, float, request.urlopen`.
**CN:** 辅助函数 `_fetch_metrics` 封装了与 `fetch metrics` 相关的可复用逻辑。 输入参数：`host, port`。 关键调用包括 `read.decode, _METRIC_RE.finditer, urlopen.read, m.group, float, request.urlopen`。

### _fetch_decode_metrics (lines 92-93)
```python
def _fetch_decode_metrics() -> dict[str, float]:
    return _fetch_metrics(DECODE_HOST, DECODE_PORT)
```
**EN:** Helper function `_fetch_decode_metrics` encapsulates reusable logic for `fetch decode metrics`. Key calls include `_fetch_metrics`.
**CN:** 辅助函数 `_fetch_decode_metrics` 封装了与 `fetch decode metrics` 相关的可复用逻辑。 关键调用包括 `_fetch_metrics`。

### _fetch_prefill_metrics (lines 96-97)
```python
def _fetch_prefill_metrics() -> dict[str, float]:
    return _fetch_metrics(PREFILL_HOST, PREFILL_PORT)
```
**EN:** Helper function `_fetch_prefill_metrics` encapsulates reusable logic for `fetch prefill metrics`. Key calls include `_fetch_metrics`.
**CN:** 辅助函数 `_fetch_prefill_metrics` 封装了与 `fetch prefill metrics` 相关的可复用逻辑。 关键调用包括 `_fetch_metrics`。

### Module state / 模块级状态 (line 100)
```python
_NIXL_BYTES_RE = re.compile(r"vllm:nixl_bytes_transferred_sum\b.*?\s+([\d.eE+\-]+)")
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_NIXL_BYTES_RE`. Shared setup calls include `re.compile`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_NIXL_BYTES_RE`。 共享初始化调用包括 `re.compile`。

### _fetch_nixl_bytes (lines 103-109)
```python
def _fetch_nixl_bytes(host: str, port: str) -> float:
    """Scrape total NIXL bytes transferred from a vLLM server."""
    body = urllib.request.urlopen(f"http://{host}:{port}/metrics").read().decode()
    total = 0.0
    for m in _NIXL_BYTES_RE.finditer(body):
        total += float(m.group(1))
    return total
```
**EN:** Helper function `_fetch_nixl_bytes` encapsulates reusable logic for `fetchNIXL bytes`. Inputs: `host, port`. Key calls include `read.decode, _NIXL_BYTES_RE.finditer, float, urlopen.read, m.group, request.urlopen`.
**CN:** 辅助函数 `_fetch_nixl_bytes` 封装了与 `fetchnixl bytes` 相关的可复用逻辑。 输入参数：`host, port`。 关键调用包括 `read.decode, _NIXL_BYTES_RE.finditer, float, urlopen.read, m.group, request.urlopen`。

### Module state / 模块级状态 (lines 112-115)
```python
_OFFLOAD_BYTES_RE = re.compile(
    r'vllm:kv_offload_total_bytes_total\{.*?transfer_type="([^"]+)".*?\}\s+'
    r"([\d.eE+\-]+)"
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_OFFLOAD_BYTES_RE`. Shared setup calls include `re.compile`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_OFFLOAD_BYTES_RE`。 共享初始化调用包括 `re.compile`。

### _fetch_offload_bytes (lines 118-126)
```python
def _fetch_offload_bytes(host: str, port: str) -> dict[str, float]:
    """Scrape kv_offload_total_bytes counters (CPU_to_GPU / GPU_to_CPU)."""
    body = urllib.request.urlopen(f"http://{host}:{port}/metrics").read().decode()
    result = {"CPU_to_GPU": 0.0, "GPU_to_CPU": 0.0}
    for m in _OFFLOAD_BYTES_RE.finditer(body):
        transfer_type, val = m.group(1), float(m.group(2))
        if transfer_type in result:
            result[transfer_type] += val
    return result
```
**EN:** Helper function `_fetch_offload_bytes` encapsulates reusable logic for `fetch offload bytes`. Inputs: `host, port`. Key calls include `read.decode, _OFFLOAD_BYTES_RE.finditer, urlopen.read, m.group, float, request.urlopen`.
**CN:** 辅助函数 `_fetch_offload_bytes` 封装了与 `fetch offload bytes` 相关的可复用逻辑。 输入参数：`host, port`。 关键调用包括 `read.decode, _OFFLOAD_BYTES_RE.finditer, urlopen.read, m.group, float, request.urlopen`。

### _metrics_delta (lines 129-130)
```python
def _metrics_delta(before: dict, after: dict) -> dict[str, float]:
    return {k: after.get(k, 0) - before.get(k, 0) for k in before}
```
**EN:** Helper function `_metrics_delta` encapsulates reusable logic for `metrics delta`. Inputs: `before, after`. Key calls include `after.get, before.get`.
**CN:** 辅助函数 `_metrics_delta` 封装了与 `metrics delta` 相关的可复用逻辑。 输入参数：`before, after`。 关键调用包括 `after.get, before.get`。

### Module state / 模块级状态 (lines 135-141)
```python
SHORT_PROMPT = "Red Hat is "

MEDIUM_PROMPT = (
    "Red Hat is the best company in the world to work for because it works "
    "on open source software, which means that all the contributions are "
    "delivered to the community. As a result,"
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `SHORT_PROMPT, MEDIUM_PROMPT`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`SHORT_PROMPT, MEDIUM_PROMPT`。

### _make_prompt (lines 144-146)
```python
def _make_prompt(n_tokens: int) -> str:
    """Build a prompt of ~n_tokens tokens (1 word ~ 1 token)."""
    return "word " * n_tokens
```
**EN:** Helper function `_make_prompt` encapsulates reusable logic for `prompt`. Inputs: `n_tokens`.
**CN:** 辅助函数 `_make_prompt` 封装了与 `prompt` 相关的可复用逻辑。 输入参数：`n_tokens`。

### Module state / 模块级状态 (lines 149-193)
```python
BLOCK_BOUNDARY_PROMPT = _make_prompt(BLOCK_SIZE)
ABOVE_BOUNDARY_PROMPT = _make_prompt(BLOCK_SIZE + 2)
MULTI_BLOCK_PROMPT = _make_prompt(BLOCK_SIZE * 4)

FULL_CACHE_HIT_PROMPT = (  # noqa: E501
    "The history of computing begins with Charles Babbage who designed the "
    "Analytical Engine in the 1830s which is considered the first general "
    "purpose computer design in history. Ada Lovelace is widely regarded as "
    "the first computer programmer for her work on the Analytical Engine. "
    "The modern era of computing began with Alan Turing who formalized the "
    "concept of computation with his Turing machine in 1936. During World "
    "War Two Turing worked at Bletchley Park to break the Enigma cipher. "
    "After the war the first electronic computers were built including ENIAC "
    "at the University of Pennsylvania and Colossus at Bletchley Park. "
    "These early machines filled entire rooms and used vacuum tubes for logic. "
    "The invention of the transistor at Bell Labs in 1947 revolutionized "
    "computing by making smaller and more reliable machines possible. "
    "The integrated circuit followed in the late 1950s combining multiple "
# ... excerpt omitted for brevity ...
    "parts of the input sequence. Large language models like GPT and BERT "
    "demonstrated that pre-training on massive text corpora followed by fine "
    "tuning on specific tasks could achieve state of the art results across "
    "a wide range of benchmarks. Scaling laws suggest that larger models "
    "trained on more data continue to improve in capability."
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `BLOCK_BOUNDARY_PROMPT, ABOVE_BOUNDARY_PROMPT, MULTI_BLOCK_PROMPT, FULL_CACHE_HIT_PROMPT, PARTIAL_CACHE_PREFIX, PARTIAL_CACHE_EXTENDED`. Shared setup calls include `_make_prompt`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`BLOCK_BOUNDARY_PROMPT, ABOVE_BOUNDARY_PROMPT, MULTI_BLOCK_PROMPT, FULL_CACHE_HIT_PROMPT, PARTIAL_CACHE_PREFIX, PARTIAL_CACHE_EXTENDED`。 共享初始化调用包括 `_make_prompt`。

### test_short_prompt_correctness (lines 203-222)
```python
def test_short_prompt_correctness():
    """Short prompt (< block_size): output matches prefill, NIXL used."""
    n0 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    m0 = _fetch_decode_metrics()
    proxy_text, _ = _complete(proxy_client, SHORT_PROMPT)
    time.sleep(1)
    m1 = _fetch_decode_metrics()
    n1 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    d = _metrics_delta(m0, m1)

    prefill_text, _ = _complete(prefill_client, SHORT_PROMPT)
    print(f"SHORT PROMPT: {proxy_text=}, nixl_bytes_delta={n1 - n0}")
    assert proxy_text == prefill_text
    assert d["external_kv_transfer"] > 0, (
        "NIXL transfer did not occur — decode may have silently fallen back "
        "to local compute"
    )
    assert n1 - n0 > 0, (
        f"expected nixl_bytes_transferred to increase, got delta={n1 - n0}"
    )
```
**EN:** Test case covering `short prompt correctness`. It exercises `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `short prompt correctness` 的测试用例。 该测试会调用 `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`。 代码主体包含 3 个显式断言。

### test_block_boundary_correctness (lines 225-244)
```python
def test_block_boundary_correctness():
    """Exactly block_size tokens: output matches prefill, NIXL used."""
    n0 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    m0 = _fetch_decode_metrics()
    proxy_text, pt = _complete(proxy_client, BLOCK_BOUNDARY_PROMPT)
    time.sleep(1)
    m1 = _fetch_decode_metrics()
    n1 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    d = _metrics_delta(m0, m1)

    prefill_text, _ = _complete(prefill_client, BLOCK_BOUNDARY_PROMPT)
    print(f"BLOCK BOUNDARY: {pt} prompt tokens, nixl_bytes_delta={n1 - n0}")
    assert proxy_text == prefill_text
    assert d["external_kv_transfer"] > 0, (
        "NIXL transfer did not occur — decode may have silently fallen back "
        "to local compute"
    )
    assert n1 - n0 > 0, (
        f"expected nixl_bytes_transferred to increase, got delta={n1 - n0}"
    )
```
**EN:** Test case covering `block boundary correctness`. It exercises `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `block boundary correctness` 的测试用例。 该测试会调用 `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`。 代码主体包含 3 个显式断言。

### test_above_block_boundary_correctness (lines 247-266)
```python
def test_above_block_boundary_correctness():
    """Just above block_size (partial second block): output matches prefill."""
    n0 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    m0 = _fetch_decode_metrics()
    proxy_text, pt = _complete(proxy_client, ABOVE_BOUNDARY_PROMPT)
    time.sleep(1)
    m1 = _fetch_decode_metrics()
    n1 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    d = _metrics_delta(m0, m1)

    prefill_text, _ = _complete(prefill_client, ABOVE_BOUNDARY_PROMPT)
    print(f"ABOVE BOUNDARY: {pt} prompt tokens, nixl_bytes_delta={n1 - n0}")
    assert proxy_text == prefill_text
    assert d["external_kv_transfer"] > 0, (
        "NIXL transfer did not occur — decode may have silently fallen back "
        "to local compute"
    )
    assert n1 - n0 > 0, (
        f"expected nixl_bytes_transferred to increase, got delta={n1 - n0}"
    )
```
**EN:** Test case covering `above block boundary correctness`. It exercises `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `above block boundary correctness` 的测试用例。 该测试会调用 `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`。 代码主体包含 3 个显式断言。

### test_multi_block_correctness (lines 269-288)
```python
def test_multi_block_correctness():
    """Multi-block prompt (~4x block_size): output matches prefill."""
    n0 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    m0 = _fetch_decode_metrics()
    proxy_text, pt = _complete(proxy_client, MULTI_BLOCK_PROMPT)
    time.sleep(1)
    m1 = _fetch_decode_metrics()
    n1 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    d = _metrics_delta(m0, m1)

    prefill_text, _ = _complete(prefill_client, MULTI_BLOCK_PROMPT)
    print(f"MULTI BLOCK: {pt} prompt tokens, nixl_bytes_delta={n1 - n0}")
    assert proxy_text == prefill_text
    assert d["external_kv_transfer"] > 0, (
        "NIXL transfer did not occur — decode may have silently fallen back "
        "to local compute"
    )
    assert n1 - n0 > 0, (
        f"expected nixl_bytes_transferred to increase, got delta={n1 - n0}"
    )
```
**EN:** Test case covering `multi block correctness`. It exercises `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `multi block correctness` 的测试用例。 该测试会调用 `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`。 代码主体包含 3 个显式断言。

### test_cold_decode_no_cache_hit_metrics (lines 299-323)
```python
def test_cold_decode_no_cache_hit_metrics():
    """Cold decode: external_kv_transfer==P, local_cache_hit==0, local_compute==0."""
    n0 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    m0 = _fetch_decode_metrics()
    proxy_text, P = _complete(proxy_client, MEDIUM_PROMPT)
    time.sleep(1)
    m1 = _fetch_decode_metrics()
    n1 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    d = _metrics_delta(m0, m1)

    print(f"COLD DECODE: {P} prompt tokens, metrics delta: {d}")
    print(f"  nixl_bytes_delta={n1 - n0}")
    assert len(proxy_text) > 0, "proxy returned empty response"
    assert d["external_kv_transfer"] == P, (
        f"expected external_kv_transfer={P}, got {d['external_kv_transfer']}"
    )
    assert d["local_compute"] == 0, (
        f"expected local_compute=0, got {d['local_compute']}"
    )
    assert d["local_cache_hit"] == 0, (
        f"expected local_cache_hit=0, got {d['local_cache_hit']}"
    )
    assert n1 - n0 > 0, (
        f"expected nixl_bytes_transferred to increase, got delta={n1 - n0}"
    )
```
**EN:** Test case covering `cold decode no cache hit metrics`. It exercises `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `cold decode no 缓存 hit metrics` 的测试用例。 该测试会调用 `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`。 代码主体包含 5 个显式断言。

### test_full_decode_gpu_cache_hit_metrics (lines 326-357)
```python
def test_full_decode_gpu_cache_hit_metrics():
    """Prime decode, resend via proxy: local_cache_hit==cached blocks."""
    decode_text, _ = _complete(decode_client, FULL_CACHE_HIT_PROMPT)

    n0 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    m0 = _fetch_decode_metrics()
    proxy_text, P = _complete(proxy_client, FULL_CACHE_HIT_PROMPT)
    time.sleep(1)
    m1 = _fetch_decode_metrics()
    n1 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    d = _metrics_delta(m0, m1)

    cached = (P // BLOCK_SIZE) * BLOCK_SIZE
    expected_nixl = P - cached

    print(f"FULL CACHE HIT: {P} tokens, cached={cached}, nixl={expected_nixl}")
    print(f"  metrics delta: {d}, nixl_bytes_delta={n1 - n0}")
    assert len(proxy_text) > 0, "proxy returned empty response"
    assert d["local_cache_hit"] == cached, (
        f"expected local_cache_hit={cached}, got {d['local_cache_hit']}"
    )
    assert d["external_kv_transfer"] == expected_nixl, (
        f"expected external_kv_transfer={expected_nixl}, "
        f"got {d['external_kv_transfer']}"
    )
    assert d["local_compute"] == 0, (
        f"expected local_compute=0, got {d['local_compute']}"
    )
    assert n1 - n0 > 0, (
        f"expected nixl_bytes_transferred to increase (partial NIXL for "
        f"uncached tail), got delta={n1 - n0}"
    )
```
**EN:** Test case covering `full decode GPU cache hit metrics`. It exercises `_complete, _fetch_nixl_bytes, _fetch_decode_metrics, time.sleep, _metrics_delta, print`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `full decode gpu 缓存 hit metrics` 的测试用例。 该测试会调用 `_complete, _fetch_nixl_bytes, _fetch_decode_metrics, time.sleep, _metrics_delta, print`。 代码主体包含 5 个显式断言。

### test_partial_decode_gpu_cache_hit_metrics (lines 360-395)
```python
def test_partial_decode_gpu_cache_hit_metrics():
    """Prime with prefix, extend via proxy: partial local_cache_hit."""
    _, prefix_tokens = _complete(decode_client, PARTIAL_CACHE_PREFIX)
    cached = (prefix_tokens // BLOCK_SIZE) * BLOCK_SIZE
    assert cached >= BLOCK_SIZE, (
        f"PARTIAL_CACHE_PREFIX too short ({prefix_tokens} tokens) for partial "
        f"cache hit test with block_size={BLOCK_SIZE}"
    )

    n0 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    m0 = _fetch_decode_metrics()
    proxy_text, P = _complete(proxy_client, PARTIAL_CACHE_EXTENDED)
    time.sleep(1)
    m1 = _fetch_decode_metrics()
    n1 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    d = _metrics_delta(m0, m1)
    expected_nixl = P - cached
    # ... excerpt omitted for brevity ...
    print(f"PARTIAL CACHE HIT: {P} tokens, cached={cached}, nixl={expected_nixl}")
    assert len(proxy_text) > 0, "proxy returned empty response"
    assert d["external_kv_transfer"] == expected_nixl, (
        f"expected external_kv_transfer={expected_nixl}, "
    assert d["local_cache_hit"] == cached, (
        f"expected local_cache_hit={cached}, got {d['local_cache_hit']}"
        f"expected local_compute=0, got {d['local_compute']}"
    assert n1 - n0 > 0, (
        f"expected nixl_bytes_transferred to increase (NIXL for uncached "
        f"tail), got delta={n1 - n0}"
```
**EN:** Test case covering `partial decode GPU cache hit metrics`. It exercises `_complete, _fetch_nixl_bytes, _fetch_decode_metrics, time.sleep, _metrics_delta, print`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `partial decode gpu 缓存 hit metrics` 的测试用例。 该测试会调用 `_complete, _fetch_nixl_bytes, _fetch_decode_metrics, time.sleep, _metrics_delta, print`。 代码主体包含 6 个显式断言。

### test_decode_direct_all_local_compute (lines 398-420)
```python
def test_decode_direct_all_local_compute():
    """Direct decode (no proxy): local_compute==P, no transfers."""
    prompt = "The speed of light is approximately"
    n0 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    m0 = _fetch_decode_metrics()
    text, P = _complete(decode_client, prompt)
    time.sleep(1)
    m1 = _fetch_decode_metrics()
    n1 = _fetch_nixl_bytes(DECODE_HOST, DECODE_PORT)
    d = _metrics_delta(m0, m1)

    print(f"DIRECT DECODE: {text!r} ({P} tokens), metrics delta: {d}")
    print(f"  nixl_bytes_delta={n1 - n0}")
    assert len(text.strip()) > 0, "empty output from direct decode"
    assert d["local_compute"] == P, (
        f"expected local_compute={P}, got {d['local_compute']}"
    )
    assert d["external_kv_transfer"] == 0, (
        f"expected external_kv_transfer=0, got {d['external_kv_transfer']}"
    )
    assert n1 - n0 == 0, (
        f"expected no nixl_bytes_transferred for direct decode, got delta={n1 - n0}"
    )
```
**EN:** Test case covering `decode direct all local compute`. It exercises `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `decode direct all local compute` 的测试用例。 该测试会调用 `_fetch_nixl_bytes, _fetch_decode_metrics, _complete, time.sleep, _metrics_delta, print`。 代码主体包含 4 个显式断言。

### Module state / 模块级状态 (lines 432-445)
```python
EVICTION_PROMPT = (  # noqa: E501
    "Quantum computing leverages quantum mechanical phenomena like "
    "superposition and entanglement to perform computations that would be "
    "intractable for classical computers. This has implications for "
    "cryptography, drug discovery, and optimization problems. Richard Feynman "
    "first proposed the idea of quantum computing in 1982 when he observed "
    "that simulating quantum systems on classical computers was exponentially "
    "hard. Peter Shor developed a quantum algorithm for factoring large "
    "numbers in polynomial time which threatens RSA encryption. Grover search "
    "algorithm provides a quadratic speedup for unstructured search problems. "
    "Companies like IBM Google and Rigetti are building quantum processors "
    "with increasing numbers of qubits. Error correction remains a major "
    "challenge as quantum states are extremely fragile and prone to decoherence."
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `EVICTION_PROMPT`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`EVICTION_PROMPT`。

### test_prefill_cpu_offload_after_gpu_eviction (lines 448-477)
```python
def test_prefill_cpu_offload_after_gpu_eviction():
    """Prefill-side: evict GPU, re-request directly, CPU offload restores KV."""
    text1, P = _complete(prefill_client, EVICTION_PROMPT, max_tokens=30)

    for i in range(100):
        _complete(prefill_client, f"Eviction prompt number {i}: " + _make_prompt(200))

    ob0 = _fetch_offload_bytes(PREFILL_HOST, PREFILL_PORT)
    m0 = _fetch_prefill_metrics()
    text2, _ = _complete(prefill_client, EVICTION_PROMPT, max_tokens=30)

    cpu_to_gpu_delta = 0.0
    for _ in range(10):
        time.sleep(1)
        ob1 = _fetch_offload_bytes(PREFILL_HOST, PREFILL_PORT)
        cpu_to_gpu_delta = ob1["CPU_to_GPU"] - ob0["CPU_to_GPU"]
        if cpu_to_gpu_delta > 0:
            break

    m1 = _fetch_prefill_metrics()
    d = _metrics_delta(m0, m1)

    print(f"PREFILL CPU OFFLOAD: run1={text1[:60]!r}, run2={text2[:60]!r}")
    print(f"  prefill metrics delta: {d}")
    print(f"  cpu_to_gpu bytes delta: {cpu_to_gpu_delta}")
    assert text1 == text2, f"inconsistent after eviction: {text1=!r}, {text2=!r}"
    assert cpu_to_gpu_delta > 0, (
        f"expected cpu_to_gpu bytes > 0 (OffloadingConnector should restore "
        f"KV from CPU to GPU), got {cpu_to_gpu_delta}"
    )
```
**EN:** Test case covering `prefill CPU offload after GPU eviction`. It exercises `_complete, range, _fetch_offload_bytes, _fetch_prefill_metrics, _metrics_delta, print`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill cpu offload after gpu eviction` 的测试用例。 该测试会调用 `_complete, range, _fetch_offload_bytes, _fetch_prefill_metrics, _metrics_delta, print`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `openai, regex`.
- **CN:** 外部库：`openai, regex`。
- **EN:** Standard-library support: `os, time, urllib.request`.
- **CN:** 标准库支持：`os, time, urllib.request`。
