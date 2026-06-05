# test_prefill_delayer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/scheduler/test_prefill_delayer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates prefill delayer behavior in SGLang's scheduler area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 scheduler 领域中与 prefill delayer 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import os
import re
import time
import unittest
from dataclasses import dataclass
from types import SimpleNamespace
from typing import List, Optional

import openai
import requests
import torch

from sglang.bench_serving import run_benchmark
from sglang.srt.managers.prefill_delayer import PrefillDelayer
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MLA_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    get_benchmark_args,
    popen_launch_server,
    run_distributed_test,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `os`, `re`, `time`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `os`, `re`, `time`。

### Lines 29-36: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(
    est_time=300,
    stage="base-c",
    runner_config="8-gpu-h200",
    disabled="Temporarily disabled",
)

WORLD_SIZE = os.environ.get("SGLANG_TEST_WORLD_SIZE", "8")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, get.
**CN:** 该代码块通过 register_cuda_ci, get 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 37-40: supporting source context / 辅助源码上下文
```python

# ============================ Unit Tests ============================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 41-42: class NegotiateCall declaration / 类 NegotiateCall 声明
```python
@dataclass
class NegotiateCall:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 43-54: class-level constants and configuration for `NegotiateCall` / 类级常量与配置
```python
    prefillable: List[bool]
    token_usage: List[float]
    # Optional scheduler state; when None, _run_negotiate_test does not pass
    # the kwarg and the delayer falls back to the historical behavior of
    # reading kwargs.get(..., 0).
    running_batch: Optional[List[int]] = None
    max_prefill_bs: Optional[List[int]] = None
    waiting_queue_len: Optional[List[int]] = None
    max_running_requests: Optional[int] = None
    # Inter-call sleep (seconds). Used to exercise the queue-trigger
    # wall-clock timeout.
    sleep_before_s: float = 0.0
```
**EN:** This block defines shared names such as `prefillable`, `token_usage`, `running_batch`, `max_prefill_bs`, `waiting_queue_len`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `prefillable`, `token_usage`, `running_batch`, `max_prefill_bs`, `waiting_queue_len` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 57-58: class NegotiateTestCase declaration / 类 NegotiateTestCase 声明
```python
@dataclass
class NegotiateTestCase:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 59-68: class-level constants and configuration for `NegotiateTestCase` / 类级常量与配置
```python
    name: str
    max_delay_passes: int
    token_usage_low_watermark: Optional[float]
    calls: List[NegotiateCall]
    expected_allow: bool
    expected_reason: str
    # Queue-trigger knobs (new in the queue-based delayer). Leave both None
    # to exercise the legacy slot-only code paths.
    queue_min_ratio: Optional[float] = None
    max_delay_ms: Optional[float] = None
```
**EN:** This block defines shared names such as `name`, `max_delay_passes`, `token_usage_low_watermark`, `calls`, `expected_allow`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `name`, `max_delay_passes`, `token_usage_low_watermark`, `calls`, `expected_allow` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 71-114: function run negotiate test / 函数 run negotiate test
```python
def _run_negotiate_test(rank, test_cases):
    world_size = torch.distributed.get_world_size()
    cpu_group = torch.distributed.new_group(backend="gloo")

    for case in test_cases:
        delayer = PrefillDelayer(
            dp_size=world_size,
            attn_tp_size=1,
            cpu_group=cpu_group,
            server_args=SimpleNamespace(
                enable_dp_attention=True,
                disaggregation_mode="null",
                disable_overlap_schedule=False,
                prefill_delayer_queue_min_ratio=case.queue_min_ratio,
                prefill_delayer_max_delay_ms=case.max_delay_ms,
            ),
            max_delay_passes=case.max_delay_passes,
            token_usage_low_watermark=case.token_usage_low_watermark,
        )

        for call in case.calls:
            if call.sleep_before_s > 0:
                time.sleep(call.sleep_before_s)

            extra_kwargs = {}
            if call.running_batch is not None:
                extra_kwargs["running_batch"] = call.running_batch[rank]
            if call.max_prefill_bs is not None:
                extra_kwargs["max_prefill_bs"] = call.max_prefill_bs[rank]
            if call.waiting_queue_len is not None:
                extra_kwargs["waiting_queue_len"] = call.waiting_queue_len[rank]
            if call.max_running_requests is not None:
                extra_kwargs["max_running_requests"] = call.max_running_requests

            result = delayer._negotiate_should_allow_prefill(
                local_prefillable=call.prefillable[rank],
                token_usage=call.token_usage[rank],
                **extra_kwargs,
            )

        assert (result.output_allow, result.output_reason) == (
            case.expected_allow,
            case.expected_reason,
        ), f"Case {case.name} rank {rank}"
```
**EN:** This block implements `_run_negotiate_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_negotiate_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 117-196: module-level constants and configuration (part 1/3) / 模块级常量与配置（第 1/3 部分）
```python
_NEGOTIATE_TEST_CASES = [
    NegotiateTestCase(
        name="all_prefillable",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        calls=[
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
            )
        ],
        expected_allow=True,
        expected_reason="no_wait",
    ),
    NegotiateTestCase(
        name="all_prefillable_with_previous_wait",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        calls=[
            NegotiateCall(
                prefillable=[True, False, True, False],
                token_usage=[0.9, 0.9, 0.9, 0.9],
            ),
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
            ),
        ],
        expected_allow=True,
        expected_reason="wait_success",
    ),
    NegotiateTestCase(
        name="none_prefillable",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        calls=[
            NegotiateCall(
                prefillable=[False, False, False, False],
                token_usage=[0.9, 0.9, 0.9, 0.9],
            )
        ],
        expected_allow=True,
        expected_reason="",
    ),
    NegotiateTestCase(
        name="mixed_delay",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        calls=[
            NegotiateCall(
                prefillable=[True, False, True, False],
                token_usage=[0.9, 0.9, 0.9, 0.9],
            )
        ],
        expected_allow=False,
        expected_reason="delay",
    ),
    NegotiateTestCase(
        name="mixed_watermark_force_allow",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        calls=[
            NegotiateCall(
                prefillable=[True, False, True, False],
                token_usage=[0.5, 0.9, 0.9, 0.9],
            )
        ],
        expected_allow=True,
        expected_reason="token_watermark",
    ),
    NegotiateTestCase(
        name="mixed_watermark_disabled",
        max_delay_passes=100,
        token_usage_low_watermark=None,
        calls=[
            NegotiateCall(
                prefillable=[True, False, True, False],
                token_usage=[0.5, 0.9, 0.9, 0.9],
            )
        ],
```
**EN:** This block defines shared names such as `_NEGOTIATE_TEST_CASES`. These values centralize configuration that later tests or helpers reuse. This chunk continues part 1 of the same logical block.
**CN:** 该代码块定义了 `_NEGOTIATE_TEST_CASES` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。 这一段对应同一逻辑块的第 1 部分。

### Lines 197-276: module-level constants and configuration (part 2/3) / 模块级常量与配置（第 2/3 部分）
```python
        expected_allow=False,
        expected_reason="delay",
    ),
    NegotiateTestCase(
        name="mixed_watermark_not_prefillable",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        calls=[
            NegotiateCall(
                prefillable=[False, False, True, False],
                token_usage=[0.5, 0.9, 0.9, 0.9],
            )
        ],
        expected_allow=False,
        expected_reason="delay",
    ),
    NegotiateTestCase(
        name="mixed_timeout",
        max_delay_passes=3,
        token_usage_low_watermark=0.8,
        calls=[
            NegotiateCall(
                prefillable=[True, False, True, False],
                token_usage=[0.9, 0.9, 0.9, 0.9],
            ),
            NegotiateCall(
                prefillable=[True, False, True, False],
                token_usage=[0.9, 0.9, 0.9, 0.9],
            ),
            NegotiateCall(
                prefillable=[True, False, True, False],
                token_usage=[0.9, 0.9, 0.9, 0.9],
            ),
        ],
        expected_allow=True,
        expected_reason="wait_timeout",
    ),
    # Queue-based trigger: waiting queue below queue_min = min(running * R,
    # max_prefill_bs) should defer prefill. With R=0.5, running=100 and
    # max_prefill_bs=80, queue_min = min(50, 80) = 50, and queue_len=10 < 50.
    NegotiateTestCase(
        name="queue_trigger_delay",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        queue_min_ratio=0.5,
        max_delay_ms=5000,
        calls=[
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
                running_batch=[100, 100, 100, 100],
                max_prefill_bs=[80, 80, 80, 80],
                waiting_queue_len=[10, 10, 10, 10],
                max_running_requests=1024,
            ),
            # skip_first_delayer consumes the first would-be delay; a second
            # identical call must actually delay.
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
                running_batch=[100, 100, 100, 100],
                max_prefill_bs=[80, 80, 80, 80],
                waiting_queue_len=[10, 10, 10, 10],
                max_running_requests=1024,
            ),
        ],
        expected_allow=False,
        expected_reason="delay",
    ),
    # Waiting queue at or above queue_min: queue trigger must not fire.
    NegotiateTestCase(
        name="queue_trigger_above_threshold",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        queue_min_ratio=0.5,
        max_delay_ms=5000,
        calls=[
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
```
**EN:** This block defines shared names such as `_NEGOTIATE_TEST_CASES`. These values centralize configuration that later tests or helpers reuse. This chunk continues part 2 of the same logical block.
**CN:** 该代码块定义了 `_NEGOTIATE_TEST_CASES` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。 这一段对应同一逻辑块的第 2 部分。

### Lines 277-350: module-level constants and configuration (part 3/3) / 模块级常量与配置（第 3/3 部分）
```python
                running_batch=[100, 100, 100, 100],
                max_prefill_bs=[80, 80, 80, 80],
                waiting_queue_len=[64, 64, 64, 64],
                max_running_requests=1024,
            )
        ],
        expected_allow=True,
        expected_reason="no_wait",
    ),
    # queue_min_ratio unset: queue trigger is opt-in and must stay disabled
    # even when running_batch and queue_len would otherwise trigger it.
    NegotiateTestCase(
        name="queue_trigger_disabled_when_ratio_unset",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        queue_min_ratio=None,
        max_delay_ms=None,
        calls=[
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
                running_batch=[100, 100, 100, 100],
                max_prefill_bs=[80, 80, 80, 80],
                waiting_queue_len=[1, 1, 1, 1],
                max_running_requests=1024,
            )
        ],
        expected_allow=True,
        expected_reason="no_wait",
    ),
    # max_delay_ms wall-clock timeout: once a single queue-trigger delay
    # exceeds the cap, prefill must be force-released.
    # Call sequence:
    #   1) queue_condition holds but skip_first_delayer consumes it
    #      (no state recorded, falls through to allow)
    #   2) queue_condition holds -> delay, records start_time in state
    #   3) after sleeping past max_delay_ms, elapsed >= cap -> force release
    NegotiateTestCase(
        name="queue_trigger_wall_clock_timeout",
        max_delay_passes=100,
        token_usage_low_watermark=0.8,
        queue_min_ratio=0.5,
        max_delay_ms=50,
        calls=[
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
                running_batch=[100, 100, 100, 100],
                max_prefill_bs=[80, 80, 80, 80],
                waiting_queue_len=[10, 10, 10, 10],
                max_running_requests=1024,
            ),
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
                running_batch=[100, 100, 100, 100],
                max_prefill_bs=[80, 80, 80, 80],
                waiting_queue_len=[10, 10, 10, 10],
                max_running_requests=1024,
            ),
            NegotiateCall(
                prefillable=[True, True, True, True],
                token_usage=[0.9, 0.9, 0.9, 0.9],
                running_batch=[100, 100, 100, 100],
                max_prefill_bs=[80, 80, 80, 80],
                waiting_queue_len=[10, 10, 10, 10],
                max_running_requests=1024,
                sleep_before_s=0.2,  # > max_delay_ms (50ms)
            ),
        ],
        expected_allow=True,
        expected_reason="wait_success",
    ),
]
```
**EN:** This block defines shared names such as `_NEGOTIATE_TEST_CASES`. These values centralize configuration that later tests or helpers reuse. This chunk continues part 3 of the same logical block.
**CN:** 该代码块定义了 `_NEGOTIATE_TEST_CASES` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。 这一段对应同一逻辑块的第 3 部分。

### Lines 353-353: class TestPrefillDelayerNegotiate declaration / 类 TestPrefillDelayerNegotiate 声明
```python
class TestPrefillDelayerNegotiate(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 354-360: test case negotiate / 测试用例 negotiate
```python
    def test_negotiate(self):
        run_distributed_test(
            _run_negotiate_test,
            world_size=4,
            backend="gloo",
            test_cases=_NEGOTIATE_TEST_CASES,
        )
```
**EN:** This test exercises `test_negotiate` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negotiate`。

### Lines 361-365: supporting source context / 辅助源码上下文
```python


# ============================ E2E Tests ============================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 366-366: class TestPrefillDelayerThroughputOnlineServing declaration / 类 TestPrefillDelayerThroughputOnlineServing 声明
```python
class TestPrefillDelayerThroughputOnlineServing(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 367-390: test case throughput comparison / 测试用例 throughput comparison
```python
    def test_throughput_comparison(self):
        _run_throughput_comparison(
            self,
            test_name="online_serving",
            other_launch_args=[
                # Not really needed, only to test support non-FCFS algorithms
                "--schedule-policy",
                "lpm",
            ],
            other_benchmark_args=dict(
                num_prompts=500,
                random_input_len=30000,
                random_output_len=256,
                request_rate=32,
            ),
            # TODO: re-enable a throughput-improvement assertion once a
            # workload that reliably exercises PrefillDelayer in online-
            # serving mode is available. The current workload yields run-
            # to-run noise on H200, while the offline test below shows the
            # same code path is healthy (improvement ~+27%). We still
            # validate functionality (server boot, benchmark completion,
            # metrics emission).
            min_improvement_pct=None,
        )
```
**EN:** This test exercises `test_throughput_comparison` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_throughput_comparison`。

### Lines 393-393: class TestPrefillDelayerThroughputOfflineGen declaration / 类 TestPrefillDelayerThroughputOfflineGen 声明
```python
class TestPrefillDelayerThroughputOfflineGen(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 394-406: test case throughput comparison / 测试用例 throughput comparison
```python
    def test_throughput_comparison(self):
        _run_throughput_comparison(
            self,
            test_name="offline_gen",
            other_launch_args=["--max-total-tokens", "200000"],
            other_benchmark_args=dict(
                num_prompts=800,
                random_input_len=30000,
                random_output_len=500,
            ),
            token_usage_low_watermark=0.8,
            min_improvement_pct=20,
        )
```
**EN:** This test exercises `test_throughput_comparison` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_throughput_comparison`。

### Lines 409-432: function run throughput comparison / 函数 run throughput comparison
```python
def _run_throughput_comparison(
    test_case,
    test_name: str,
    other_launch_args,
    other_benchmark_args,
    min_improvement_pct: Optional[float],
    token_usage_low_watermark: float = None,
):
    common_kwargs = dict(
        debug_name=test_name,
        other_launch_args=other_launch_args,
        other_benchmark_args=other_benchmark_args,
        token_usage_low_watermark=token_usage_low_watermark,
    )
    res_enabled = _run_throughput_test(prefill_delayer=True, **common_kwargs)
    res_disabled = _run_throughput_test(prefill_delayer=False, **common_kwargs)

    _assert_throughput_improvement(
        test_case,
        test_name=test_name,
        res_enabled=res_enabled,
        res_disabled=res_disabled,
        min_improvement_pct=min_improvement_pct,
    )
```
**EN:** This block implements `_run_throughput_comparison` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_throughput_comparison`，承担模块行为中的一个聚焦逻辑片段。

### Lines 435-471: function run throughput test / 函数 run throughput test
```python
def _run_throughput_test(
    debug_name: str,
    prefill_delayer: bool,
    other_launch_args,
    other_benchmark_args,
    token_usage_low_watermark: float = None,
):
    model = "Qwen/Qwen3-0.6B"
    base_url = DEFAULT_URL_FOR_TEST

    process = _launch_server(
        prefill_delayer=prefill_delayer,
        model=model,
        base_url=base_url,
        other_args=other_launch_args,
        token_usage_low_watermark=token_usage_low_watermark,
    )

    try:
        args = get_benchmark_args(
            base_url=base_url,
            dataset_name="random",
            tokenizer=model,
            **other_benchmark_args,
        )
        res = run_benchmark(args)
        _print_prefill_delayer_metrics(base_url, expect_metrics=prefill_delayer)
    finally:
        kill_process_tree(process.pid)

    print(f"=== {debug_name} ({prefill_delayer=}) ===")
    res["total_throughput"] = res["input_throughput"] + res["output_throughput"]
    print(f"Input throughput: {res['input_throughput']:.2f} token/s")
    print(f"Output throughput: {res['output_throughput']:.2f} token/s")
    print(f"Total throughput: {res['total_throughput']:.2f} token/s")

    return res
```
**EN:** This block implements `_run_throughput_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_throughput_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 474-504: function assert throughput improvement / 函数 assert throughput improvement
```python
def _assert_throughput_improvement(
    test_case,
    test_name: str,
    res_enabled: dict,
    res_disabled: dict,
    min_improvement_pct: Optional[float],
):
    test_case.assertEqual(
        WORLD_SIZE,
        "8",
        f"This test requires 8 GPUs to properly measure throughput improvement, got {WORLD_SIZE}",
    )

    enabled = res_enabled["total_throughput"]
    disabled = res_disabled["total_throughput"]
    improvement_pct = (enabled - disabled) / disabled * 100

    print(f"\n=== {test_name} Throughput Comparison ===")
    print(
        f"Total: enabled={enabled:.2f}, disabled={disabled:.2f}, improvement={improvement_pct:.2f}%"
    )

    if min_improvement_pct is None:
        # Functionality-only mode: skip the perf assertion.
        return

    test_case.assertGreaterEqual(
        improvement_pct,
        min_improvement_pct,
        f"{test_name}: Throughput improvement ({improvement_pct:.2f}%) < {min_improvement_pct}%",
    )
```
**EN:** This block implements `_assert_throughput_improvement` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_throughput_improvement`，承担模块行为中的一个聚焦逻辑片段。

### Lines 507-507: class TestPrefillDelayerTokenUsageLowWatermark declaration / 类 TestPrefillDelayerTokenUsageLowWatermark 声明
```python
class TestPrefillDelayerTokenUsageLowWatermark(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 508-510: test case 1 with low watermark / 测试用例 1 with low watermark
```python
    def test_1_with_low_watermark(self):
        # The kv cache size here is deliberately small, thus we use smaller token usage
        self._run(token_usage_low_watermark=0.5)
```
**EN:** This test exercises `test_1_with_low_watermark` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_1_with_low_watermark`。

### Lines 511-513: supporting source context / 辅助源码上下文
```python

    # TODO: re-enable once sglang/sglang#22511 (DP-attention detokenizer
    # hang on H200 in CI) is fixed.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 514-516: test case 2 without low watermark / 测试用例 2 without low watermark
```python
    @unittest.skip("blocked by sgl-project/sglang#22511")
    def test_2_without_low_watermark(self):
        self._run(token_usage_low_watermark=None)
```
**EN:** This test exercises `test_2_without_low_watermark` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_2_without_low_watermark`。

### Lines 518-588: method run / 方法 run
```python
    def _run(self, token_usage_low_watermark):
        model = "Qwen/Qwen3-0.6B"
        base_url = DEFAULT_URL_FOR_TEST
        world_size = int(WORLD_SIZE)

        process = _launch_server(
            model=model,
            base_url=base_url,
            prefill_delayer=True,
            other_args=["--max-total-tokens", "50000"],
            # e.g. gen throughput is 370 tok/s on H200.
            # Will need a different threshold on B200
            max_delay_passes=3000,
            token_usage_low_watermark=token_usage_low_watermark,
        )

        async def run_test():
            client = openai.AsyncClient(base_url=f"{base_url}/v1", api_key="EMPTY")
            long_prompt = "Hello " * 5000

            async def send_blocking_request():
                return await client.chat.completions.create(
                    model=model,
                    messages=[{"role": "user", "content": long_prompt}],
                    max_tokens=10000,
                    extra_body={"data_parallel_rank": 0},
                )

            async def send_normal_request(dp_rank, req_idx):
                start = time.time()
                await client.chat.completions.create(
                    model=model,
                    messages=[{"role": "user", "content": "Say hi"}],
                    max_tokens=10,
                    extra_body={"data_parallel_rank": dp_rank},
                )
                elapsed = time.time() - start
                return dp_rank, req_idx, elapsed

            asyncio.create_task(send_blocking_request())
            await asyncio.sleep(3)

            num_reqs_per_rank = 10
            results = await asyncio.gather(
                *[
                    send_normal_request(dp_rank, req_idx)
                    for dp_rank in range(1, world_size)
                    for req_idx in range(num_reqs_per_rank)
                ]
            )

            enabled = token_usage_low_watermark is not None
            thresh = 5
            for dp_rank, req_idx, elapsed in results:
                print(f"DP rank {dp_rank} req {req_idx} completed in {elapsed:.2f}s")
                self.assertTrue(
                    (elapsed < thresh) if enabled else (elapsed > thresh),
                    f"DP rank {dp_rank} req {req_idx}: elapsed={elapsed:.2f}s, thresh={thresh}, enabled={enabled}. "
                    f"Maybe you need a different `max_delay_passes` when using hardware other than H200.",
                )

        try:
            asyncio.run(run_test())

            metrics_text = _print_prefill_delayer_metrics(base_url, expect_metrics=True)
            if token_usage_low_watermark is not None:
                total = _sum_prometheus_metric_values(metrics_text, "token_watermark")
                self.assertGreater(total, 0, "Expected token_watermark > 0")
                print(f"total token_watermark: {total}")
        finally:
            kill_process_tree(process.pid)
```
**EN:** This block implements `_run` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run`，承担模块行为中的一个聚焦逻辑片段。

### Lines 591-591: class TestPrefillDelayerAccuracy declaration / 类 TestPrefillDelayerAccuracy 声明
```python
class TestPrefillDelayerAccuracy(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 592-593: test case 1 gsm8k has prefill delayer / 测试用例 1 gsm8k has prefill delayer
```python
    def test_1_gsm8k_has_prefill_delayer(self):
        self._run_accuracy_test(prefill_delayer=True)
```
**EN:** This test exercises `test_1_gsm8k_has_prefill_delayer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_1_gsm8k_has_prefill_delayer`。

### Lines 595-596: test case 2 gsm8k no prefill delayer / 测试用例 2 gsm8k no prefill delayer
```python
    def test_2_gsm8k_no_prefill_delayer(self):
        self._run_accuracy_test(prefill_delayer=False)
```
**EN:** This test exercises `test_2_gsm8k_no_prefill_delayer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_2_gsm8k_no_prefill_delayer`。

### Lines 598-627: method run accuracy test / 方法 run accuracy test
```python
    def _run_accuracy_test(self, prefill_delayer: bool):
        model = DEFAULT_MLA_MODEL_NAME_FOR_TEST
        base_url = DEFAULT_URL_FOR_TEST
        process = _launch_server(
            prefill_delayer=prefill_delayer,
            model=model,
            base_url=base_url,
            other_args=[
                # Not really needed, only to test support non-FCFS algorithms
                "--schedule-policy",
                "lpm",
                # Use this to ensure prefill delayer will be run
                "--max-total-tokens",
                "4096",
            ],
        )
        try:
            args = SimpleNamespace(
                base_url=base_url,
                model=model,
                eval_name="gsm8k",
                num_examples=None,
                num_threads=1024,
            )
            metrics = run_eval(args)
            print(f"=== gsm8k ({prefill_delayer=}) ===")
            print(f"{metrics=}")
            self.assertGreater(metrics["score"], 0.57)
        finally:
            kill_process_tree(process.pid)
```
**EN:** This block implements `_run_accuracy_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_accuracy_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 630-670: function launch server / 函数 launch server
```python
def _launch_server(
    *,
    model,
    base_url,
    prefill_delayer: bool,
    other_args,
    max_delay_passes: int = 100,
    token_usage_low_watermark: float = None,
):
    os.environ["SGLANG_PREFILL_DELAYER_DEBUG_LOG"] = "1"

    return popen_launch_server(
        model,
        base_url,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        other_args=[
            "--trust-remote-code",
            "--tp",
            WORLD_SIZE,
            "--enable-dp-attention",
            "--dp",
            WORLD_SIZE,
            "--chunked-prefill-size",
            "131072",
            "--mem-fraction-static",
            "0.6",
            "--enable-metrics",
            *(["--enable-prefill-delayer"] if prefill_delayer else []),
            "--prefill-delayer-max-delay-passes",
            str(max_delay_passes),
            *(
                [
                    "--prefill-delayer-token-usage-low-watermark",
                    str(token_usage_low_watermark),
                ]
                if token_usage_low_watermark is not None
                else []
            ),
            *(other_args or []),
        ],
    )
```
**EN:** This block implements `_launch_server` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_launch_server`，承担模块行为中的一个聚焦逻辑片段。

### Lines 673-687: function print prefill delayer metrics / 函数 print prefill delayer metrics
```python
def _print_prefill_delayer_metrics(base_url: str, expect_metrics: bool) -> str:
    metrics_response = requests.get(f"{base_url}/metrics")
    assert metrics_response.status_code == 200
    metrics_text = metrics_response.text
    prefill_delayer_metrics = [
        line for line in metrics_text.split("\n") if "prefill_delayer" in line
    ]
    print("=== PrefillDelayer Metrics ===")
    for line in prefill_delayer_metrics:
        print(line)
    if expect_metrics:
        assert "sglang:prefill_delayer_wait_forward_passes" in metrics_text
        assert "sglang:prefill_delayer_wait_seconds" in metrics_text
        assert "sglang:prefill_delayer_outcomes_total" in metrics_text
    return metrics_text
```
**EN:** This block implements `_print_prefill_delayer_metrics` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_print_prefill_delayer_metrics`，承担模块行为中的一个聚焦逻辑片段。

### Lines 690-692: function sum prometheus metric values / 函数 sum prometheus metric values
```python
def _sum_prometheus_metric_values(metrics_text: str, label_value: str) -> int:
    matches = re.findall(rf'{label_value}".*?\}} (\d+)', metrics_text)
    return sum(int(m) for m in matches)
```
**EN:** This block implements `_sum_prometheus_metric_values` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_sum_prometheus_metric_values`，承担模块行为中的一个聚焦逻辑片段。

### Lines 695-696: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `NegotiateCall`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `NegotiateTestCase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_run_negotiate_test`: This block implements `_run_negotiate_test` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_negotiate_test`，承担模块行为中的一个聚焦逻辑片段。
- `TestPrefillDelayerNegotiate`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPrefillDelayerThroughputOnlineServing`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPrefillDelayerThroughputOfflineGen`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_run_throughput_comparison`: This block implements `_run_throughput_comparison` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_throughput_comparison`，承担模块行为中的一个聚焦逻辑片段。
- `_run_throughput_test`: This block implements `_run_throughput_test` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_throughput_test`，承担模块行为中的一个聚焦逻辑片段。
- `TestPrefillDelayerNegotiate.test_negotiate`: This test exercises `test_negotiate` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negotiate`。
- `TestPrefillDelayerThroughputOnlineServing.test_throughput_comparison`: This test exercises `test_throughput_comparison` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_throughput_comparison`。
- `TestPrefillDelayerThroughputOfflineGen.test_throughput_comparison`: This test exercises `test_throughput_comparison` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_throughput_comparison`。
- `TestPrefillDelayerTokenUsageLowWatermark.test_1_with_low_watermark`: This test exercises `test_1_with_low_watermark` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_1_with_low_watermark`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `os`, `re`, `time`, `unittest`, `dataclasses`, `types`, `typing`
- **Third-party modules / 第三方模块**: `openai`, `requests`, `torch`
- **Internal modules / 内部模块**: `sglang.bench_serving`, `sglang.srt.managers.prefill_delayer`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 696
