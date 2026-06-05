# test_session_latency.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/sessions/test_session_latency.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates session latency behavior in SGLang's sessions area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 sessions 领域中与 session latency 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: supporting statements / 辅助语句
```python
"""
Benchmark: Streaming Session Inter-Turn Latency

Tests:
  1. Latency (bs=8):    regular vs streaming, assert speedup >= 2x
  2. Correctness (bs=1): regular vs streaming, assert output equal + speedup
  3. Random lengths (bs=8): streaming only, random input/output lens, no crash

Usage:
    python -m pytest test_session_latency.py -s
    python -m unittest test_session_latency.BenchSessionLatency
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 14-32: module imports and dependencies / 模块导入与依赖
```python
import random
import time
import unittest
from concurrent.futures import ThreadPoolExecutor
from dataclasses import dataclass, field
from typing import Dict, List, Optional

import requests
from tabulate import tabulate

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `random`, `time`, `unittest`, `concurrent.futures`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `random`, `time`, `unittest`, `concurrent.futures`。

### Lines 34-60: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=122, stage="extra-a", runner_config="1-gpu-large")

NUM_TURNS = 150
INPUT_LEN = 16
GEN_LEN = 8
NUM_CONCURRENT = 8
TAIL_TURNS = 10
SAMPLE_TURNS = 8

NUM_TURNS_RANDOM = 50
RANDOM_INPUT_LEN_RANGE = (8, 64)
RANDOM_OUTPUT_LEN_RANGE = (4, 32)

FILLER_TEXT = (
    "The quick brown fox jumps over the lazy dog. "
    "Pack my box with five dozen liquor jugs. "
    "How vexingly quick daft zebras jump. "
    "Sphinx of black quartz, judge my vow. "
) * 200

SAMPLING_PARAMS = {
    "temperature": 0,
    "max_new_tokens": GEN_LEN,
    "no_stop_trim": True,
    "skip_special_tokens": False,
    "ignore_eos": True,
}
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 63-64: class TurnResult declaration / 类 TurnResult 声明
```python
@dataclass
class TurnResult:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 65-71: class-level constants and configuration for `TurnResult` / 类级常量与配置
```python
    turn: int
    context_len: int
    cached_tokens: int
    prompt_tokens: int
    completion_tokens: int
    client_latency_ms: float
    e2e_latency_ms: float
```
**EN:** This block defines shared names such as `turn`, `context_len`, `cached_tokens`, `prompt_tokens`, `completion_tokens`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `turn`, `context_len`, `cached_tokens`, `prompt_tokens`, `completion_tokens` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 74-75: class ModeResult declaration / 类 ModeResult 声明
```python
@dataclass
class ModeResult:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 76-78: class-level constants and configuration for `ModeResult` / 类级常量与配置
```python
    mode: str
    turns: List[TurnResult] = field(default_factory=list)
    outputs: List[str] = field(default_factory=list)
```
**EN:** This block defines shared names such as `mode`, `turns`, `outputs`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `mode`, `turns`, `outputs` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 79-85: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Helpers
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 86-105: function generate input chunks / 函数 generate input chunks
```python
def _generate_input_chunks(
    tokenizer, num_turns: int, input_len: int, offset: int = 0
) -> List[List[int]]:
    all_ids = tokenizer.encode(FILLER_TEXT)
    if all_ids and all_ids[0] == tokenizer.bos_token_id:
        all_ids = all_ids[1:]

    start = offset * num_turns * input_len
    needed = start + num_turns * input_len
    while len(all_ids) < needed:
        all_ids = all_ids + all_ids
    chunks = [
        all_ids[start + i * input_len : start + (i + 1) * input_len]
        for i in range(num_turns)
    ]

    if tokenizer.bos_token_id is not None:
        chunks[0] = [tokenizer.bos_token_id] + chunks[0]

    return chunks
```
**EN:** This block implements `_generate_input_chunks` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_generate_input_chunks`，承担模块行为中的一个聚焦逻辑片段。

### Lines 108-135: function generate random input chunks / 函数 generate random input chunks
```python
def _generate_random_input_chunks(
    tokenizer,
    num_turns: int,
    min_len: int,
    max_len: int,
    rng: random.Random,
    offset: int = 0,
) -> List[List[int]]:
    all_ids = tokenizer.encode(FILLER_TEXT)
    if all_ids and all_ids[0] == tokenizer.bos_token_id:
        all_ids = all_ids[1:]

    total_max = offset * num_turns * max_len + num_turns * max_len
    while len(all_ids) < total_max:
        all_ids = all_ids + all_ids

    chunks: List[List[int]] = []
    pos = offset * num_turns * max_len
    for i in range(num_turns):
        length = rng.randint(min_len, max_len)
        chunk = all_ids[pos : pos + length]
        pos += length
        chunks.append(chunk)

    if tokenizer.bos_token_id is not None:
        chunks[0] = [tokenizer.bos_token_id] + chunks[0]

    return chunks
```
**EN:** This block implements `_generate_random_input_chunks` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_generate_random_input_chunks`，承担模块行为中的一个聚焦逻辑片段。

### Lines 138-142: function send generate / 函数 send generate
```python
def _send_generate(base_url: str, payload: dict) -> dict:
    resp = requests.post(base_url + "/generate", json=payload)
    if resp.status_code != 200:
        raise RuntimeError(f"Generate failed ({resp.status_code}): {resp.text}")
    return resp.json()
```
**EN:** This block implements `_send_generate` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_send_generate`，承担模块行为中的一个聚焦逻辑片段。

### Lines 145-156: function record turn / 函数 record turn
```python
def _record_turn(
    turn_idx: int, context_len: int, meta: dict, client_latency_ms: float
) -> TurnResult:
    return TurnResult(
        turn=turn_idx + 1,
        context_len=context_len,
        cached_tokens=meta["cached_tokens"],
        prompt_tokens=meta["prompt_tokens"],
        completion_tokens=meta["completion_tokens"],
        client_latency_ms=client_latency_ms,
        e2e_latency_ms=meta.get("e2e_latency", 0) * 1000,
    )
```
**EN:** This block implements `_record_turn` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_record_turn`，承担模块行为中的一个聚焦逻辑片段。

### Lines 157-163: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Single-session runner (called by worker threads)
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 164-215: function run one session / 函数 run one session
```python
def _run_one_session(
    base_url: str,
    chunks: List[List[int]],
    streaming: bool = False,
    per_turn_gen_lens: Optional[List[int]] = None,
) -> ModeResult:
    mode = "streaming_session" if streaming else "regular_session"
    result = ModeResult(mode=mode)

    default_gen = GEN_LEN
    if per_turn_gen_lens is not None:
        max_gen = max(per_turn_gen_lens)
    else:
        max_gen = default_gen
    capacity = sum(len(c) for c in chunks) + len(chunks) * max_gen + 1024

    open_payload: dict = {"capacity_of_str_len": capacity}
    if streaming:
        open_payload["streaming"] = True
    session_id = requests.post(base_url + "/open_session", json=open_payload).json()

    rid = None
    context_len = 0

    for turn_idx, chunk_ids in enumerate(chunks):
        context_len += len(chunk_ids)

        if per_turn_gen_lens is not None:
            sp = {**SAMPLING_PARAMS, "max_new_tokens": per_turn_gen_lens[turn_idx]}
        else:
            sp = SAMPLING_PARAMS

        t0 = time.perf_counter()
        response = _send_generate(
            base_url,
            {
                "input_ids": chunk_ids,
                "session_params": {"id": session_id, "rid": rid},
                "sampling_params": sp,
            },
        )
        client_lat = (time.perf_counter() - t0) * 1000

        meta = response["meta_info"]
        rid = meta["id"]
        context_len += meta["completion_tokens"]

        result.turns.append(_record_turn(turn_idx, context_len, meta, client_lat))
        result.outputs.append(response["text"])

    requests.post(base_url + "/close_session", json={"session_id": session_id})
    return result
```
**EN:** This block implements `_run_one_session` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_one_session`，承担模块行为中的一个聚焦逻辑片段。

### Lines 216-222: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Stats & reporting
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 223-232: function collect latencies / 函数 collect latencies
```python
def _collect_latencies(
    results: List[ModeResult], last_n: Optional[int] = None
) -> List[float]:
    lats = []
    for r in results:
        turns = r.turns[1:]  # skip turn 1
        if last_n is not None:
            turns = r.turns[-last_n:]
        lats.extend(t.client_latency_ms for t in turns)
    return lats
```
**EN:** This block implements `_collect_latencies` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_collect_latencies`，承担模块行为中的一个聚焦逻辑片段。

### Lines 235-236: function avg / 函数 avg
```python
def _avg(values: List[float]) -> float:
    return sum(values) / len(values) if values else 0.0
```
**EN:** This block implements `_avg` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_avg`，承担模块行为中的一个聚焦逻辑片段。

### Lines 239-270: function print mode table / 函数 print mode table
```python
def _print_mode_table(result: ModeResult, label: str = ""):
    tag = f"{result.mode} ({label})" if label else result.mode
    print(f"\n  [{tag}]  {len(result.turns)} turns")

    n = len(result.turns)
    if n <= SAMPLE_TURNS * 2:
        indices = list(range(n))
    else:
        indices = list(range(SAMPLE_TURNS)) + [-1] + list(range(n - SAMPLE_TURNS, n))

    rows = []
    for idx in indices:
        if idx == -1:
            rows.append(["..."] * 5)
            continue
        t = result.turns[idx]
        rows.append(
            [
                t.turn,
                t.context_len,
                t.cached_tokens,
                f"{t.client_latency_ms:.1f}ms",
                f"{t.e2e_latency_ms:.1f}ms",
            ]
        )
    print(
        tabulate(
            rows,
            headers=["Turn", "Context", "Cached", "Client Lat", "E2E Lat"],
            colalign=("right",) * 5,
        )
    )
```
**EN:** This block implements `_print_mode_table` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_print_mode_table`，承担模块行为中的一个聚焦逻辑片段。

### Lines 273-308: function print summary / 函数 print summary
```python
def _print_summary(all_results: Dict[str, List[ModeResult]]):
    stats = [
        (
            mode,
            _avg(_collect_latencies(rs)),
            _avg(_collect_latencies(rs, last_n=TAIL_TURNS)),
        )
        for mode, rs in all_results.items()
    ]
    base_all, base_tail = (stats[0][1] or 1.0), (stats[0][2] or 1.0)
    tail_label = f"last {TAIL_TURNS}"

    print(f"\n  SUMMARY  ({NUM_CONCURRENT} sessions x {NUM_TURNS} turns)")
    rows = [
        [
            mode,
            f"{a:.1f}ms",
            f"{t:.1f}ms",
            f"{base_all / a:.2f}x" if a else "inf",
            f"{base_tail / t:.2f}x" if t else "inf",
        ]
        for mode, a, t in stats
    ]
    print(
        tabulate(
            rows,
            headers=[
                "Mode",
                "Avg (all)",
                f"Avg ({tail_label})",
                "Speedup (all)",
                f"Speedup ({tail_label})",
            ],
            colalign=("left", "right", "right", "right", "right"),
        )
    )
```
**EN:** This block implements `_print_summary` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_print_summary`，承担模块行为中的一个聚焦逻辑片段。

### Lines 309-315: supporting source context / 辅助源码上下文
```python


# ---------------------------------------------------------------------------
# Test class
# ---------------------------------------------------------------------------


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 316-316: class TestSessionLatency declaration / 类 TestSessionLatency 声明
```python
class TestSessionLatency(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 317-349: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "openai/gpt-oss-20b"
        cls.base_url = DEFAULT_URL_FOR_TEST
        # NOTE: Overlap scheduling commits KV cache one step ahead,
        # so the last decode token is cached (unlike non-overlap).
        # Disable overlap to keep session cache behavior consistent.
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--disable-overlap-schedule",
                "--enable-streaming-session",
                "--mem-fraction-static",
                "0.70",
                "--disable-piecewise-cuda-graph",
                "--page-size",
                "4",
            ],
        )
        cls.tokenizer = get_tokenizer(cls.model)

        requests.post(cls.base_url + "/flush_cache")
        _send_generate(
            cls.base_url,
            {
                "input_ids": cls.tokenizer.encode("Hello world"),
                "sampling_params": {"temperature": 0, "max_new_tokens": 1},
            },
        )

        cls.all_results: Dict[str, List[ModeResult]] = {}
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 351-355: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if len(cls.all_results) > 1:
            _print_summary(cls.all_results)
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 357-392: method run concurrent session / 方法 run concurrent session
```python
    def _run_concurrent_session(
        self,
        streaming: bool = False,
        num_concurrent: int = NUM_CONCURRENT,
        num_turns: int = NUM_TURNS,
        input_len: int = INPUT_LEN,
        per_turn_gen_lens: Optional[List[int]] = None,
        random_input_chunks: bool = False,
        rng: Optional[random.Random] = None,
    ) -> List[ModeResult]:
        requests.post(self.base_url + "/flush_cache")

        def run_one(session_idx):
            if random_input_chunks and rng is not None:
                per_session_rng = random.Random(rng.randint(0, 2**32) + session_idx)
                chunks = _generate_random_input_chunks(
                    self.tokenizer,
                    num_turns,
                    RANDOM_INPUT_LEN_RANGE[0],
                    RANDOM_INPUT_LEN_RANGE[1],
                    per_session_rng,
                    offset=session_idx,
                )
            else:
                chunks = _generate_input_chunks(
                    self.tokenizer, num_turns, input_len, offset=session_idx
                )
            return _run_one_session(
                self.base_url,
                chunks,
                streaming=streaming,
                per_turn_gen_lens=per_turn_gen_lens,
            )

        with ThreadPoolExecutor(max_workers=num_concurrent) as pool:
            return list(pool.map(run_one, range(num_concurrent)))
```
**EN:** This block implements `_run_concurrent_session` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_concurrent_session`，承担模块行为中的一个聚焦逻辑片段。

### Lines 393-397: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------
    # Test methods (alphabetical order matters for dependencies)
    # ------------------------------------------------------------------

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 398-402: test case regular session / 测试用例 regular session
```python
    def test_regular_session(self):
        """Run regular (non-streaming) sessions for latency baseline."""
        results = self._run_concurrent_session(streaming=False)
        self.__class__.all_results["regular_session"] = results
        _print_mode_table(results[0], label="session 0")
```
**EN:** Run regular (non-streaming) sessions for latency baseline. This test exercises `test_regular_session` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Run regular (non-streaming) sessions for latency baseline. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_regular_session`。

### Lines 404-420: test case streaming session / 测试用例 streaming session
```python
    def test_streaming_session(self):
        """Latency test: bs=8, assert streaming >= 2x faster than regular."""
        results = self._run_concurrent_session(streaming=True)
        self.__class__.all_results["streaming_session"] = results
        _print_mode_table(results[0], label="session 0")

        reg_list = self.__class__.all_results.get("regular_session")
        if reg_list:
            reg_tail = _avg(_collect_latencies(reg_list, last_n=TAIL_TURNS))
            stm_tail = _avg(_collect_latencies(results, last_n=TAIL_TURNS))
            speedup = reg_tail / stm_tail if stm_tail > 0 else float("inf")
            self.assertGreaterEqual(
                speedup,
                1.4,
                f"streaming should be >=1.4x faster on last {TAIL_TURNS} turns "
                f"(regular={reg_tail:.1f}ms, streaming={stm_tail:.1f}ms, speedup={speedup:.2f}x)",
            )
```
**EN:** Latency test: bs=8, assert streaming >= 2x faster than regular. This test exercises `test_streaming_session` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Latency test: bs=8, assert streaming >= 2x faster than regular. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_session`。

### Lines 422-442: test case streaming session correctness / 测试用例 streaming session correctness
```python
    def test_streaming_session_correctness(self):
        """Correctness test: bs=1, assert output equal + latency speedup."""
        correctness_turns = 30
        reg = self._run_concurrent_session(
            streaming=False, num_concurrent=1, num_turns=correctness_turns
        )
        stm = self._run_concurrent_session(
            streaming=True, num_concurrent=1, num_turns=correctness_turns
        )

        _print_mode_table(reg[0], label="correctness regular")
        _print_mode_table(stm[0], label="correctness streaming")

        reg_out = reg[0].outputs
        stm_out = stm[0].outputs
        mismatches = sum(1 for a, b in zip(reg_out, stm_out) if a != b)
        self.assertEqual(
            mismatches,
            0,
            f"regular vs streaming (bs=1): {mismatches}/{len(reg_out)} turns differ",
        )
```
**EN:** Correctness test: bs=1, assert output equal + latency speedup. This test exercises `test_streaming_session_correctness` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Correctness test: bs=1, assert output equal + latency speedup. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_session_correctness`。

### Lines 444-465: test case streaming session random lengths / 测试用例 streaming session random lengths
```python
    def test_streaming_session_random_lengths(self):
        """Stress test: bs=8, streaming only, random input/output lens."""
        rng = random.Random(42)
        gen_lens = [
            rng.randint(*RANDOM_OUTPUT_LEN_RANGE) for _ in range(NUM_TURNS_RANDOM)
        ]

        results = self._run_concurrent_session(
            streaming=True,
            num_turns=NUM_TURNS_RANDOM,
            per_turn_gen_lens=gen_lens,
            random_input_chunks=True,
            rng=random.Random(42),
        )

        for i, r in enumerate(results):
            self.assertEqual(
                len(r.turns),
                NUM_TURNS_RANDOM,
                f"session {i}: expected {NUM_TURNS_RANDOM} turns, got {len(r.turns)}",
            )
        _print_mode_table(results[0], label="random streaming session 0")
```
**EN:** Stress test: bs=8, streaming only, random input/output lens. This test exercises `test_streaming_session_random_lengths` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Stress test: bs=8, streaming only, random input/output lens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_session_random_lengths`。

### Lines 468-469: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TurnResult`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `ModeResult`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_generate_input_chunks`: This block implements `_generate_input_chunks` and captures one focused piece of the module's behavior. / 该代码块实现 `_generate_input_chunks`，承担模块行为中的一个聚焦逻辑片段。
- `_generate_random_input_chunks`: This block implements `_generate_random_input_chunks` and captures one focused piece of the module's behavior. / 该代码块实现 `_generate_random_input_chunks`，承担模块行为中的一个聚焦逻辑片段。
- `_send_generate`: This block implements `_send_generate` and captures one focused piece of the module's behavior. / 该代码块实现 `_send_generate`，承担模块行为中的一个聚焦逻辑片段。
- `_record_turn`: This block implements `_record_turn` and captures one focused piece of the module's behavior. / 该代码块实现 `_record_turn`，承担模块行为中的一个聚焦逻辑片段。
- `_run_one_session`: This block implements `_run_one_session` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_one_session`，承担模块行为中的一个聚焦逻辑片段。
- `_collect_latencies`: This block implements `_collect_latencies` and captures one focused piece of the module's behavior. / 该代码块实现 `_collect_latencies`，承担模块行为中的一个聚焦逻辑片段。
- `TestSessionLatency.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSessionLatency.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestSessionLatency._run_concurrent_session`: This block implements `_run_concurrent_session` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_concurrent_session`，承担模块行为中的一个聚焦逻辑片段。
- `TestSessionLatency.test_regular_session`: Run regular (non-streaming) sessions for latency baseline. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_regular_session`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `time`, `unittest`, `concurrent.futures`, `dataclasses`, `typing`
- **Third-party modules / 第三方模块**: `requests`, `tabulate`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 469
