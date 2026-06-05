# test_online_batch_invariance.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/determinism/test_online_batch_invariance.py`
- **Repository**: vllm-project/vllm
- **Purpose**: HTTP-based batch invariance test: send requests to a running vLLM server and compare BS=1 vs BS=N results (tokens and per-step logprobs). / 该文件的文档字符串表明其用途：`http-based 批处理 invariance test: send requests to a running vllm server and compare bs=1 vs bs=n results (tokens and per-step logprobs)`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-11)
```python
"""
HTTP-based batch invariance test: send requests to a running
vLLM server and compare BS=1 vs BS=N results (tokens and per-step logprobs).

Environment variables:
  - VLLM_TEST_MODEL: served model name (e.g., Qwen/Qwen3-1.7B / DeepSeek-R1)
  - VLLM_TP_SIZE: tensor parallelism size (e.g., 4)

"""
```
**EN:** Module docstring that declares the scope of the file: HTTP-based batch invariance test: send requests to a running vLLM server and compare BS=1 vs BS=N results (tokens and per-step logprobs).
**CN:** 模块文档字符串直接说明了文件范围：`http-based 批处理 invariance test: send requests to a running vllm server and compare bs=1 vs bs=n results (tokens and per-step logprobs)`。

### Imports and setup / 导入与设置 (lines 13-22)
```python
import os
import random
import sys
from typing import Any

import openai
import pytest
from utils import BACKENDS, TEST_MODEL, _random_prompt, skip_unsupported

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `openai, pytest, utils`. Local helpers come from `tests.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `openai, pytest, utils`。 本地测试辅助逻辑来自 `tests.utils`。

### _request_completion (lines 25-50)
```python
def _request_completion(
    client: openai.OpenAI,
    model: str,
    prompt: Any,
    sp: dict[str, Any],
    max_retries: int = 3,
    retry_backoff: float = 0.5,
) -> dict[str, Any] | None:
    payload: dict[str, Any] = {"model": model, "prompt": prompt}
    payload.update(sp)

    for attempt in range(max_retries + 1):
        try:
            completion = client.completions.create(**payload)
            # Convert to plain dict so downstream logic can keep using
            # dict-style access just like with raw HTTP JSON.
            return completion.model_dump()
        except Exception as e:  # pragma: no cover
            if attempt < max_retries:
                import time as _t

                _t.sleep(retry_backoff * (2**attempt))
                continue
            sys.stderr.write(f"Error: {e}\n")
            return None
    return None
```
**EN:** Helper function `_request_completion` encapsulates reusable logic for `request completion`. Inputs: `client, model, prompt, sp, max_retries, retry_backoff`. Key calls include `payload.update, range, completions.create, completion.model_dump, stderr.write, _t.sleep`.
**CN:** 辅助函数 `_request_completion` 封装了与 `request completion` 相关的可复用逻辑。 输入参数：`client, model, prompt, sp, max_retries, retry_backoff`。 关键调用包括 `payload.update, range, completions.create, completion.model_dump, stderr.write, _t.sleep`。

### _extract_tokens_and_logprobs (lines 53-62)
```python
def _extract_tokens_and_logprobs(
    choice: dict[str, Any],
) -> tuple[list[Any], list[float] | None]:
    tokens: list[Any] = []
    token_logprobs: list[float] | None = None
    lp = choice.get("logprobs")
    if lp and isinstance(lp, dict):
        tokens = lp.get("token_ids") or lp.get("tokens") or []
        token_logprobs = lp.get("token_logprobs", None)
    return tokens, token_logprobs
```
**EN:** Helper function `_extract_tokens_and_logprobs` encapsulates reusable logic for `extract tokens and logprobs`. Inputs: `choice`. Key calls include `choice.get, isinstance, lp.get`.
**CN:** 辅助函数 `_extract_tokens_and_logprobs` 封装了与 `extract tokens and 对数概率` 相关的可复用逻辑。 输入参数：`choice`。 关键调用包括 `choice.get, isinstance, lp.get`。

### _compare_bs1_vs_bsn_single_process (lines 65-133)
```python
def _compare_bs1_vs_bsn_single_process(
    prompts: list[str],
    sp_kwargs: dict[str, Any],
    client: openai.OpenAI,
    model_name: str,
) -> None:
    # BS=1
    bs1_tokens_per_prompt: list[list[Any]] = []
    bs1_logprobs_per_prompt: list[list[float] | None] = []
    for p in prompts:
        resp = _request_completion(client, model_name, p, sp_kwargs)
        if resp is None or not resp.get("choices"):
            raise AssertionError("BS=1 empty/failed response")
        choice = resp["choices"][0]
        toks, lps = _extract_tokens_and_logprobs(choice)
        if lps is None:
            raise AssertionError(
                "logprobs not returned; ensure server supports 'logprobs'"
    # ... excerpt omitted for brevity ...
                diff = abs(a - b)
                raise AssertionError(
                    f"Prompt {i} Step {t}: Bitwise mismatch "
                    f"(abs diff={diff:.6e}). "
                    f"BS=1 tokens: {tokens_bs1} BS=N tokens: {tokens_bsN}"
                )
```
**EN:** Helper function `_compare_bs1_vs_bsn_single_process` encapsulates reusable logic for `compare bs1 vs bsn single process`. Inputs: `prompts, sp_kwargs, client, model_name`. Key calls include `_request_completion, resp.get, enumerate, _extract_tokens_and_logprobs, bs1_tokens_per_prompt.append, bs1_logprobs_per_prompt.append`.
**CN:** 辅助函数 `_compare_bs1_vs_bsn_single_process` 封装了与 `compare bs1 vs bsn single process` 相关的可复用逻辑。 输入参数：`prompts, sp_kwargs, client, model_name`。 关键调用包括 `_request_completion, resp.get, enumerate, _extract_tokens_and_logprobs, bs1_tokens_per_prompt.append, bs1_logprobs_per_prompt.append`。

### test_logprobs_bitwise_batch_invariance_bs1_vs_bsN (lines 138-168)
```python
def test_logprobs_bitwise_batch_invariance_bs1_vs_bsN(
    backend: str,
) -> None:
    random.seed(int(os.getenv("VLLM_TEST_SEED", "12345")))
    prompts_all = [_random_prompt(10, 50) for _ in range(32)]

    sp_kwargs: dict[str, Any] = {
        "temperature": 0.6,
        "top_p": 1.0,
        "max_tokens": 8,
        "seed": 42,
        "logprobs": 5,
    }

    tp_size = os.getenv("VLLM_TP_SIZE", "1")
    server_args: list[str] = [
        "--max-model-len=8192",
        "--max-num-seqs=32",
        f"--attention-backend={backend}",
    ]
    if tp_size:
        server_args += ["-tp", tp_size]

    with RemoteOpenAIServer(TEST_MODEL, server_args) as server:
        client = server.get_client()
        _compare_bs1_vs_bsn_single_process(
            prompts=prompts_all,
            sp_kwargs=sp_kwargs,
            client=client,
            model_name=TEST_MODEL,
        )
```
**EN:** Parameterized test covering `logprobs bitwise batch invariance bs1 vs bsN`. Parameter axes: `backend`. Inputs/fixtures: `backend`. It exercises `mark.parametrize, random.seed, os.getenv, int, _random_prompt, RemoteOpenAIServer`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `对数概率 bitwise 批处理 invariance bs1 vs bsn` 的测试用例。 参数维度：`backend`。 输入或 fixture：`backend`。 该测试会调用 `mark.parametrize, random.seed, os.getenv, int, _random_prompt, RemoteOpenAIServer`。 主要通过 mock、回调或输出检查来完成验证。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Token/logprob accounting correctness
- **CN:** token/logprob 统计正确性
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查

## Dependencies / 依赖关系
- **EN:** External libraries: `openai, pytest, utils`.
- **CN:** 外部库：`openai, pytest, utils`。
- **EN:** Local test helpers: `tests.utils`.
- **CN:** 本地测试辅助模块：`tests.utils`。
- **EN:** Standard-library support: `os, random, sys, typing, time`.
- **CN:** 标准库支持：`os, random, sys, typing, time`。
