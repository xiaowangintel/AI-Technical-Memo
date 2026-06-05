# test_dsv4_pro_mtp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/dsv4/test_dsv4_pro_mtp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `dsv4 pro mtp` scenario in `test/manual/dsv4`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/dsv4` 中的 `dsv4 pro mtp` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-70: Constants and scenario settings / 常量与场景配置
```python
"""DSV4-Pro 1.6T MTP performance tests on B200 TP=8.

1. TestDSV4ProMTPSimulatedAcc — `SGLANG_SIMULATE_ACC_LEN=3` pins EAGLE accept
   length so latency comparisons are apples-to-apples. Runs `bench_one_batch_server`
   at bs=1 for isl=4096 and isl=900000 (osl=1024).

2. TestDSV4ProMTPHongloumeng — real EAGLE accept (no SIMULATE) on Chinese
   long-context input (`hongloumeng.txt`, ~627k DSV4 tokens). Builds a one-line
   custom JSONL dataset on the fly and drives `bench_serving --dataset-name custom`
   with one short slice (30k tokens) and the full long prompt.

Manual test (8× B200, 1.6T weights). Not registered in CI.
"""

import json
import os
import tempfile
import unittest
from types import SimpleNamespace

import requests

from sglang.bench_one_batch_server import BenchArgs as OneBatchBenchArgs
from sglang.bench_one_batch_server import run_benchmark as run_one_batch_benchmark
from sglang.bench_serving import run_benchmark as run_serving_benchmark
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

DSV4_PRO_MODEL_PATH = "deepseek-ai/DeepSeek-V4-Pro"

HONGLOUMENG_PATH = os.environ.get(
    "SGLANG_HONGLOUMENG_PATH",
    os.path.join(os.path.dirname(__file__), "hongloumeng.txt"),
)

DSV4_PRO_BASE_ENV = {
    "SGLANG_ENABLE_SPEC_V2": "1",
    "SGLANG_OPT_USE_CUSTOM_ALL_REDUCE_V2": "1",
}

DSV4_PRO_SERVER_ARGS = [
    "--trust-remote-code",
    "--tp",
    "8",
    "--moe-runner-backend",
    "flashinfer_mxfp4",
    "--speculative-algorithm",
    "EAGLE",
    "--speculative-num-steps",
    "3",
    "--speculative-eagle-topk",
    "1",
    "--speculative-num-draft-tokens",
    "4",
    "--chunked-prefill-size",
    "4096",
    "--disable-flashinfer-autotune",
    "--mem-fraction-static",
    "0.82",
    "--max-running-requests",
    "8",
]
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `accept`, `input`, `slice` and `test`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 71-81: Helper routines around _launch_dsv4_pro_server / 辅助例程
```python
def _launch_dsv4_pro_server(extra_env=None):
    env = dict(DSV4_PRO_BASE_ENV)
    if extra_env:
        env.update(extra_env)
    return popen_launch_server(
        DSV4_PRO_MODEL_PATH,
        DEFAULT_URL_FOR_TEST,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 4,
        other_args=DSV4_PRO_SERVER_ARGS,
        env=env,
    )
```
**EN:** This range implements helper routine(s) `_launch_dsv4_pro_server` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `update` and `popen_launch_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 84-92: Class definition for TestDSV4ProMTPSimulatedAcc / 类定义
```python
class TestDSV4ProMTPSimulatedAcc(CustomTestCase):
    """bs=1 latency at isl=4096 / 900000 with `SGLANG_SIMULATE_ACC_LEN=3`.

    Reference (B200 Pro TP8):
      - isl=4096   → output 194.6 tok/s, accept 2.96
      - isl=900000 → output 174.6 tok/s, accept 2.93
    """

    @classmethod
```
**EN:** This range declares `TestDSV4ProMTPSimulatedAcc`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `Reference`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 93-97: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = _launch_dsv4_pro_server(
            extra_env={"SGLANG_SIMULATE_ACC_LEN": "3"}
        )
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `_launch_dsv4_pro_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 98-99: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 100-102: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `hasattr` and `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 104-108: Helper routines around _run_one_batch / 辅助例程
```python
    def _run_one_batch(self, input_len):
        requests.get(self.base_url + "/flush_cache")
        server_args = ServerArgs(model_path=DSV4_PRO_MODEL_PATH)
        bench_args = OneBatchBenchArgs(
            run_name=f"dsv4_pro_simacc_isl{input_len}",
```
**EN:** This range implements helper routine(s) `_run_one_batch` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`, `ServerArgs` and `OneBatchBenchArgs`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 109-121: Assertions and result checks / 断言与结果检查
```python
            batch_size=(1,),
            input_len=(input_len,),
            output_len=(1024,),
            base_url=self.base_url,
            skip_warmup=True,
            result_filename=os.path.join(
                tempfile.gettempdir(), f"dsv4_pro_simacc_isl{input_len}.jsonl"
            ),
            append_to_github_summary=False,
        )
        results, _ = run_one_batch_benchmark(server_args, bench_args)
        self.assertTrue(results, "bench_one_batch_server returned no results")
        return results[0]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `join`, `gettempdir`, `run_one_batch_benchmark` and `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 123-133: Test routines around test_isl_4096 / 测试例程
```python
    def test_isl_4096(self):
        r = self._run_one_batch(4096)
        print(
            f"[pro simacc isl=4096] output_throughput={r.output_throughput:.2f} tok/s "
            f"latency={r.latency:.2f}s last_ttft={r.last_ttft:.2f}s "
            f"acc_length={r.acc_length:.2f}"
        )
        # Reference 194.6 tok/s / acc=2.96 — give 10% throughput margin and a
        # generous accept-length floor to absorb run-to-run jitter.
        self.assertGreater(r.output_throughput, 175.0)
        self.assertGreater(r.acc_length, 2.85)
```
**EN:** This range defines concrete test routine(s) `test_isl_4096`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `_run_one_batch` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 135-144: Test routines around test_isl_900k / 测试例程
```python
    def test_isl_900k(self):
        r = self._run_one_batch(900_000)
        print(
            f"[pro simacc isl=900k] output_throughput={r.output_throughput:.2f} tok/s "
            f"latency={r.latency:.2f}s last_ttft={r.last_ttft:.2f}s "
            f"acc_length={r.acc_length:.2f}"
        )
        # Reference 174.6 tok/s / acc=2.93.
        self.assertGreater(r.output_throughput, 155.0)
        self.assertGreater(r.acc_length, 2.85)
```
**EN:** This range defines concrete test routine(s) `test_isl_900k`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `_run_one_batch` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 147-164: Helper routines around _build_hongloumeng_jsonl / 辅助例程
```python
def _build_hongloumeng_jsonl(num_tokens, tokenizer, out_path):
    """Slice the first `num_tokens` DSV4 tokens of hongloumeng.txt into a
    one-line CustomDataset JSONL. Pass num_tokens=None to keep the full text.
    """
    with open(HONGLOUMENG_PATH, "r", encoding="utf-8") as f:
        text = f.read()
    if num_tokens is not None:
        ids = tokenizer.encode(text)
        text = tokenizer.decode(ids[:num_tokens])
    with open(out_path, "w", encoding="utf-8") as f:
        f.write(
            json.dumps(
                {"conversations": [{"value": text}, {"value": "x"}]},
                ensure_ascii=False,
            )
            + "\n"
        )
    return out_path
```
**EN:** This range implements helper routine(s) `_build_hongloumeng_jsonl` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `read`, `encode`, `decode` and `write`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 167-179: Class definition for TestDSV4ProMTPHongloumeng / 类定义
```python
class TestDSV4ProMTPHongloumeng(CustomTestCase):
    """Real EAGLE accept on Chinese long-context (hongloumeng.txt).

    Reference (B200 Pro TP8, no SIMULATE):
      - isl=30000  → output 124.4 tok/s, decode peak 184 tok/s, accept 2.47
      - isl=627059 → output 125.7 tok/s, decode peak 179 tok/s, accept 2.52
    """

    SHORT_TOKENS = 30_000
    LONG_TOKENS = None  # full file (~627k DSV4 tokens)
    OUTPUT_TOKENS = 4096

    @classmethod
```
**EN:** This range declares `TestDSV4ProMTPHongloumeng`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `context`, `Reference` and `file`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 180-191: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = _launch_dsv4_pro_server()

        # Resolve tokenizer once; the server reports its own tokenizer path so
        # on-the-fly token-level slicing matches what the server will see.
        info = requests.get(cls.base_url + "/server_info", timeout=60).json()
        tokenizer_path = info.get("tokenizer_path") or DSV4_PRO_MODEL_PATH
        from sglang.srt.utils.hf_transformers_utils import get_tokenizer

        cls.tokenizer = get_tokenizer(tokenizer_path)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `_launch_dsv4_pro_server`, `get`, `json` and `get_tokenizer`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 192-202: Scenario logic / 场景逻辑
```python
        cls.tmpdir = tempfile.mkdtemp(prefix="dsv4_hongloumeng_")
        cls.short_jsonl = _build_hongloumeng_jsonl(
            cls.SHORT_TOKENS,
            cls.tokenizer,
            os.path.join(cls.tmpdir, "hongloumeng_30k.jsonl"),
        )
        cls.long_jsonl = _build_hongloumeng_jsonl(
            cls.LONG_TOKENS,
            cls.tokenizer,
            os.path.join(cls.tmpdir, "hongloumeng_full.jsonl"),
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `mkdtemp`, `_build_hongloumeng_jsonl` and `join`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 203-204: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 205-207: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `hasattr` and `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 209-216: Helper routines around _run_custom_bench / 辅助例程
```python
    def _run_custom_bench(self, dataset_path):
        requests.get(self.base_url + "/flush_cache")
        args = SimpleNamespace(
            backend="sglang",
            base_url=self.base_url,
            host=None,
            port=None,
            dataset_name="custom",
```
**EN:** This range implements helper routine(s) `_run_custom_bench` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get` and `SimpleNamespace`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 217-233: Scenario logic / 场景逻辑
```python
            dataset_path=dataset_path,
            model=None,
            tokenizer=None,
            num_prompts=1,
            sharegpt_output_len=self.OUTPUT_TOKENS,
            sharegpt_context_len=None,
            random_input_len=4096,
            random_output_len=2048,
            random_range_ratio=0.0,
            request_rate=float("inf"),
            max_concurrency=1,
            warmup_requests=0,
            flush_cache=True,
            multi=None,
            output_file=None,
            disable_tqdm=False,
            disable_stream=False,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 234-241: Scenario logic / 场景逻辑
```python
            return_logprob=False,
            return_routed_experts=False,
            seed=0,
            disable_ignore_eos=False,
            extra_request_body=None,
            apply_chat_template=False,
            profile=None,
            lora_name=None,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 242-249: Scenario logic / 场景逻辑
```python
            lora_request_distribution="uniform",
            lora_zipf_alpha=1.5,
            prompt_suffix="",
            device="cuda",
            pd_separated=False,
            ready_check_timeout_sec=0,
        )
        return run_serving_benchmark(args)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `run_serving_benchmark`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 251-261: Test routines around test_short_30k / 测试例程
```python
    def test_short_30k(self):
        res = self._run_custom_bench(self.short_jsonl)
        print(
            f"[hongloumeng 30k] output_throughput={res['output_throughput']:.2f} tok/s "
            f"accept_length={res['accept_length']:.2f} "
            f"mean_ttft_ms={res['mean_ttft_ms']:.0f} "
            f"mean_tpot_ms={res['mean_tpot_ms']:.2f}"
        )
        # Reference 124 tok/s / accept 2.47.
        self.assertGreater(res["output_throughput"], 105.0)
        self.assertGreater(res["accept_length"], 2.30)
```
**EN:** This range defines concrete test routine(s) `test_short_30k`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `_run_custom_bench` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 263-266: Test routines around test_long_full / 测试例程
```python
    def test_long_full(self):
        res = self._run_custom_bench(self.long_jsonl)
        print(
            f"[hongloumeng full] output_throughput={res['output_throughput']:.2f} tok/s "
```
**EN:** This range defines concrete test routine(s) `test_long_full`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_run_custom_bench`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 267-275: Assertions and result checks / 断言与结果检查
```python
            f"accept_length={res['accept_length']:.2f} "
            f"mean_ttft_ms={res['mean_ttft_ms']:.0f} "
            f"mean_tpot_ms={res['mean_tpot_ms']:.2f}"
        )
        # Reference 125 tok/s / accept 2.52. Cold prefill takes ~85s on 627k
        # tokens so the run is dominated by prefill, but decode steady-state
        # accept_length is the metric we care about.
        self.assertGreater(res["output_throughput"], 105.0)
        self.assertGreater(res["accept_length"], 2.30)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 276-279: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `os`, `tempfile`, `types`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.bench_one_batch_server`, `sglang.bench_serving`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.test_utils`
