# batch_spec.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/attention_benchmarks/batch_spec.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements batch behavior benchmarks, benchmark orchestration, attention benchmark flows, centered around `BatchRequest`, `_parse_size`, `parse_batch_spec`, `format_batch_spec`. / 实现与批处理行为基准、基准测试编排、注意力基准流程相关的逻辑，核心符号包括 `BatchRequest`, `_parse_size`, `parse_batch_spec`, `format_batch_spec`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-27)
```python
"""
Simplified batch specification grammar for attention benchmarks.

Grammar (underscore-separated segments):
  Format: (<count>?) q<q_len>(k?) (s<seq_len>(k?))?

  - count: Number of identical requests (optional, default=1)
  - q_len: Query length (number of new tokens)
  - seq_len: Total sequence length (optional, defaults to q_len for prefill)
  - 'k' suffix: Multiplies value by 1024

Common patterns:
  - Prefill:  q_len == seq_len  (e.g., "q2k" → 2048 new tokens, 2048 seq)
  - Decode:   q_len == 1        (e.g., "q1s1k" → 1 token, 1024 seq length)
  - Extend:   q_len < seq_len   (e.g., "q4s1k" → 4 tokens, 1024 seq length)

Examples:
  q2k              -> [(2048, 2048)]           # Prefill: 2048 tokens
  q1s1k            -> [(1, 1024)]              # Decode: 1 token, 1K sequence
  8q1s1k           -> [(1, 1024)] * 8          # 8 decode requests
  q4s1k            -> [(4, 1024)]              # 4-token extend (spec decode)
  2q1k_32q1s1k     -> [(1024, 1024)] * 2 + [(1, 1024)] * 32  # Mixed batch
  16q4s1k          -> [(4, 1024)] * 16         # 16 spec decode requests
"""
```
**EN:** The module docstring introduces Simplified batch specification grammar for attention benchmarks. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Simplified batch specification grammar for attention benchmarks 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 29-32)
```python
from collections import Counter
from dataclasses import dataclass

import regex as re
```
**EN:** This block gathers standard-library helpers such as `collections`, `dataclasses`; third-party packages such as `regex`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `collections`, `dataclasses`；第三方依赖，如 `regex`。这些依赖构成了后续基准测试逻辑的基础。

### Class `BatchRequest` (lines 36-64)
```python
class BatchRequest:
    """Represents a single request in a batch."""

    q_len: int  # Query length (number of new tokens)
    kv_len: int  # Total KV cache length

    @property
    def is_decode(self) -> bool:
        """True if this is a decode request (q_len == 1)."""
        return self.q_len == 1

    @property
    def is_prefill(self) -> bool:
        """True if this is a pure prefill (q_len == kv_len)."""
        return self.q_len == self.kv_len

    @property
    def is_extend(self) -> bool:
        """True if this is context extension (q_len > 1, kv_len > q_len)."""
        return self.q_len > 1 and self.kv_len > self.q_len

    @property
    def context_len(self) -> int:
        """Context length (KV cache - query)."""
        return self.kv_len - self.q_len

    def as_tuple(self) -> tuple[int, int]:
        """Return as (q_len, kv_len) tuple for compatibility."""
        return (self.q_len, self.kv_len)
```
**EN:** Class `BatchRequest` is the main object-oriented wrapper for this module. Its docstring says: Represents a single request in a batch. It extends `object` and organizes behavior through `is_decode`, `is_prefill`, `is_extend`, `context_len`, `as_tuple`.
**CN:** 类 `BatchRequest` 是该模块中的主要面向对象封装。文档字符串指出：Represents a single request in a batch。它继承自 `object`，并通过 `is_decode`, `is_prefill`, `is_extend`, `context_len`, `as_tuple` 组织行为。

### Method `BatchRequest.is_decode` (lines 43-45)
```python
    def is_decode(self) -> bool:
        """True if this is a decode request (q_len == 1)."""
        return self.q_len == 1
```
**EN:** `is_decode` True if this is a decode request (q_len == 1). It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `is_decode` 的职责是：True if this is a decode request (q_len == 1)。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `BatchRequest.is_prefill` (lines 48-50)
```python
    def is_prefill(self) -> bool:
        """True if this is a pure prefill (q_len == kv_len)."""
        return self.q_len == self.kv_len
```
**EN:** `is_prefill` True if this is a pure prefill (q_len == kv_len). It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `is_prefill` 的职责是：True if this is a pure prefill (q_len == kv_len)。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `BatchRequest.is_extend` (lines 53-55)
```python
    def is_extend(self) -> bool:
        """True if this is context extension (q_len > 1, kv_len > q_len)."""
        return self.q_len > 1 and self.kv_len > self.q_len
```
**EN:** `is_extend` True if this is context extension (q_len > 1, kv_len > q_len). It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `is_extend` 的职责是：True if this is context extension (q_len > 1, kv_len > q_len)。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `BatchRequest.context_len` (lines 58-60)
```python
    def context_len(self) -> int:
        """Context length (KV cache - query)."""
        return self.kv_len - self.q_len
```
**EN:** `context_len` Context length (KV cache - query). It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `context_len` 的职责是：Context length (KV cache - query)。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `_parse_size` (lines 67-70)
```python
def _parse_size(size_str: str, k_suffix: str) -> int:
    """Parse size string with optional 'k' suffix."""
    size = int(size_str)
    return size * 1024 if k_suffix == "k" else size
```
**EN:** `_parse_size` Parse size string with optional 'k' suffix. It mainly works with `size_str`, `k_suffix` and relies on `int` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_parse_size` 的职责是：Parse size string with optional 'k' suffix。 它主要处理 `size_str`, `k_suffix`，并结合 `int` 以及 结果返回 来完成这一段基准测试流程。

### Function `parse_batch_spec` (lines 73-102)
```python
def parse_batch_spec(spec: str) -> list[BatchRequest]:
    """
    Parse batch specification string into list of BatchRequest objects.

    Grammar: (<count>?) q<q_len>(k?) (s<seq_len>(k?))?

    Args:
        spec: Batch specification string (see module docstring for grammar)

    Returns:
        List of BatchRequest objects

    Raises:
        ValueError: If spec format is invalid
    """
    requests = []

    for seg in spec.split("_"):
        # Unified pattern: (<count>?) q<q_len>(k?) (s<seq_len>(k?))?
        m = re.match(r"^(?:(\d+))?q(\d+)(k?)(?:s(\d+)(k?))?$", seg)
        if m:
            cnt = int(m.group(1)) if m.group(1) else 1
            q_len = _parse_size(m.group(2), m.group(3))
            kv_len = _parse_size(m.group(4), m.group(5)) if m.group(4) else q_len
            requests.extend([BatchRequest(q_len=q_len, kv_len=kv_len)] * cnt)
            continue

        raise ValueError(f"Invalid batch spec segment: '{seg}'")

    return requests
```
**EN:** `parse_batch_spec` Parse batch specification string into list of BatchRequest objects. It mainly works with `spec` and relies on `spec.split`, `re.match`, `m.group`, `int`, `_parse_size`, `requests.extend` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `parse_batch_spec` 的职责是：Parse batch specification string into list of BatchRequest objects。 它主要处理 `spec`，并结合 `spec.split`, `re.match`, `m.group`, `int`, `_parse_size`, `requests.extend` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `format_batch_spec` (lines 105-156)
```python
def format_batch_spec(requests: list[BatchRequest]) -> str:
    """
    Format list of BatchRequest into human-readable string.

    Groups requests by type and provides counts and sizes.

    Args:
        requests: List of BatchRequest objects

    Returns:
        Formatted string describing the batch
    """
    kinds = {
        "prefill": [],
        "extend": [],
        "decode": [],
    }

    for req in requests:
        tup = (req.q_len, req.kv_len)
        if req.is_prefill:
            kinds["prefill"].append(tup)
        elif req.is_extend:
            kinds["extend"].append(tup)
        elif req.is_decode:
            kinds["decode"].append(tup)

    parts = []
    for kind in ["prefill", "extend", "decode"]:
        lst = kinds[kind]
        if not lst:
            continue

        cnt_total = len(lst)
    # ... omitted for brevity ...
            else:  # extend
                qstr = f"{q // 1024}k" if q % 1024 == 0 else str(q)
                kstr = f"{kv // 1024}k" if kv % 1024 == 0 else str(kv)
                inner.append(f"{cnt}xq{qstr}kv{kstr}")

        parts.append(f"{cnt_total} {kind} ({', '.join(inner)})")

    return ", ".join(parts)
```
**EN:** `format_batch_spec` Format list of BatchRequest into human-readable string. It mainly works with `requests` and relies on `kinds.append`, `len`, `Counter`, `ctr.items`, `str`, `inner.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `format_batch_spec` 的职责是：Format list of BatchRequest into human-readable string。 它主要处理 `requests`，并结合 `kinds.append`, `len`, `Counter`, `ctr.items`, `str`, `inner.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `reorder_for_flashinfer` (lines 159-174)
```python
def reorder_for_flashinfer(requests: list[BatchRequest]) -> list[BatchRequest]:
    """
    Reorder requests for FlashInfer: decode first, then prefill.

    FlashInfer expects decode requests before prefill requests for
    optimal performance.

    Args:
        requests: Original list of BatchRequest

    Returns:
        Reordered list with decode requests first
    """
    decodes = [r for r in requests if r.is_decode]
    non_decodes = [r for r in requests if not r.is_decode]
    return decodes + non_decodes
```
**EN:** `reorder_for_flashinfer` Reorder requests for FlashInfer: decode first, then prefill. It mainly works with `requests` and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `reorder_for_flashinfer` 的职责是：Reorder requests for FlashInfer: decode first, then prefill。 它主要处理 `requests`，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `split_by_type` (lines 177-203)
```python
def split_by_type(
    requests: list[BatchRequest],
) -> dict[str, list[BatchRequest]]:
    """
    Split requests by type for analysis.

    Args:
        requests: List of BatchRequest

    Returns:
        Dict with keys: 'decode', 'prefill', 'extend'
    """
    result = {
        "decode": [],
        "prefill": [],
        "extend": [],
    }

    for req in requests:
        if req.is_decode:
            result["decode"].append(req)
        elif req.is_prefill:
            result["prefill"].append(req)
        elif req.is_extend:
            result["extend"].append(req)

    return result
```
**EN:** `split_by_type` Split requests by type for analysis. It mainly works with `requests` and relies on `result.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `split_by_type` 的职责是：Split requests by type for analysis。 它主要处理 `requests`，并结合 `result.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `get_batch_stats` (lines 206-231)
```python
def get_batch_stats(requests: list[BatchRequest]) -> dict:
    """
    Compute statistics about a batch.

    Args:
        requests: List of BatchRequest

    Returns:
        Dict with batch statistics
    """
    by_type = split_by_type(requests)

    return {
        "total_requests": len(requests),
        "num_decode": len(by_type["decode"]),
        "num_prefill": len(by_type["prefill"]),
        "num_extend": len(by_type["extend"]),
        "total_tokens": sum(r.q_len for r in requests),
        "total_kv_cache": sum(r.kv_len for r in requests),
        "max_q_len": max((r.q_len for r in requests), default=0),
        "max_kv_len": max((r.kv_len for r in requests), default=0),
        "avg_q_len": sum(r.q_len for r in requests) / len(requests) if requests else 0,
        "avg_kv_len": (
            sum(r.kv_len for r in requests) / len(requests) if requests else 0
        ),
    }
```
**EN:** `get_batch_stats` Compute statistics about a batch. It mainly works with `requests` and relies on `split_by_type`, `len`, `sum`, `max` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_batch_stats` 的职责是：Compute statistics about a batch。 它主要处理 `requests`，并结合 `split_by_type`, `len`, `sum`, `max` 以及 结果返回 来完成这一段基准测试流程。

### Function `get_batch_type` (lines 234-268)
```python
def get_batch_type(batch_spec: str, spec_decode_threshold: int = 8) -> str:
    """
    Classify a batch spec into a type string.

    Args:
        batch_spec: Batch specification string (e.g., "q2k", "8q1s1k", "2q2k_8q1s1k")
        spec_decode_threshold: Max q_len to be considered spec-decode vs extend

    Returns:
        Type string: "prefill", "decode", "spec-decode", "extend", or "mixed (types...)"
    """
    requests = parse_batch_spec(batch_spec)

    # Classify each request
    types_present = set()
    for req in requests:
        if req.is_decode:
            types_present.add("decode")
        elif req.is_prefill:
            types_present.add("prefill")
        elif req.is_extend:
            # Distinguish spec-decode (small q_len) from extend (chunked prefill)
            if req.q_len <= spec_decode_threshold:
                types_present.add("spec-decode")
            else:
                types_present.add("extend")

    if len(types_present) == 1:
        return types_present.pop()
    elif len(types_present) > 1:
        # Sort for consistent output
        sorted_types = sorted(types_present)
        return f"mixed ({'+'.join(sorted_types)})"
    else:
        return "unknown"
```
**EN:** `get_batch_type` Classify a batch spec into a type string. It mainly works with `batch_spec`, `spec_decode_threshold` and relies on `parse_batch_spec`, `set`, `types_present.add`, `len`, `types_present.pop`, `sorted` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `get_batch_type` 的职责是：Classify a batch spec into a type string。 它主要处理 `batch_spec`, `spec_decode_threshold`，并结合 `parse_batch_spec`, `set`, `types_present.add`, `len`, `types_present.pop`, `sorted` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

## Key Concepts / 关键概念
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。

## Dependencies / 依赖关系
- **EN:** Standard library: `collections`, `dataclasses`.
- **CN:** 标准库依赖：`collections`, `dataclasses`。
- **EN:** Third-party packages: `regex`.
- **CN:** 第三方依赖：`regex`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
