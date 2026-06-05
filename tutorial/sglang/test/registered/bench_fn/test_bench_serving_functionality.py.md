# test_bench_serving_functionality.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/bench_fn/test_bench_serving_functionality.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on bench fn bench serving functionality in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 bench fn bench serving functionality 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Import dependencies
```python
import json
import tempfile
import threading
import time
import unittest
from http.server import BaseHTTPRequestHandler, HTTPServer
from pathlib import Path

from sglang.bench_serving import run_benchmark
from sglang.benchmark.utils import parse_custom_headers
from sglang.srt.constants import HEALTH_CHECK_RID_PREFIX
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    get_benchmark_args,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 22-23: Register CI metadata
```python
register_cuda_ci(est_time=300, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=300, suite="nightly-amd-1-gpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 25-26: Define module constants
```python
MODEL = "Qwen/Qwen3-0.6B"
NUM_CONVERSATIONS, NUM_TURNS = 4, 3
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 29-29: Define class TestBenchServingFunctionality
```python
class TestBenchServingFunctionality(CustomTestCase):
```
**EN:** This declaration introduces the `TestBenchServingFunctionality` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBenchServingFunctionality` 测试类，并说明它通过继承承担的职责。

### Lines 30-72: Run test: gsp multi turn
```python
    def test_gsp_multi_turn(self):
        with tempfile.TemporaryDirectory() as temp_dir:
            process = popen_launch_server(
                MODEL,
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--mem-fraction-static",
                    "0.7",
                    "--log-requests",
                    "--log-requests-level",
                    "3",
                    "--log-requests-format",
                    "json",
                    "--log-requests-target",
                    "stdout",
                    temp_dir,
                ],
            )
            try:
                args = get_benchmark_args(
                    base_url=DEFAULT_URL_FOR_TEST,
                    backend="sglang-oai-chat",
                    tokenizer=MODEL,
                    dataset_name="generated-shared-prefix",
                    num_prompts=NUM_CONVERSATIONS,
                    request_rate=float("inf"),
                    gsp_num_groups=2,
                    gsp_prompts_per_group=2,
                    gsp_system_prompt_len=64,
                    gsp_question_len=16,
                    gsp_output_len=16,
                    gsp_num_turns=NUM_TURNS,
                )
                args.warmup_requests = 0
                res = run_benchmark(args)
                self.assertEqual(res["completed"], NUM_CONVERSATIONS * NUM_TURNS)

                time.sleep(1)
                logs = "".join(f.read_text() for f in Path(temp_dir).glob("*.log"))
                self._verify_multi_turn_logs(logs)
            finally:
                kill_process_tree(process.pid)
```
**EN:** This test method exercises gsp multi turn and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 gsp multi turn 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 74-105: Define helper: verify multi turn logs
```python
    def _verify_multi_turn_logs(self, content: str):
        reqs = []
        for line in content.splitlines():
            idx = line.find("{")
            if idx == -1:
                continue
            try:
                obj = json.loads(line[idx:])
            except json.JSONDecodeError:
                continue
            if obj.get("event") != "request.finished":
                continue
            text = obj.get("obj", {}).get("text")
            rid = obj.get("rid", "")
            if text and not rid.startswith(HEALTH_CHECK_RID_PREFIX):
                reqs.append(text)

        self.assertGreaterEqual(len(reqs), NUM_CONVERSATIONS * NUM_TURNS)

        # Verify prefix relationships
        reqs_sorted = sorted(reqs, key=len)
        prefix_count = 0
        for i, text in enumerate(reqs_sorted):
            for j in range(i + 1, len(reqs_sorted)):
                if reqs_sorted[j].startswith(text):
                    prefix_count += 1
                    break

        expected = NUM_CONVERSATIONS * (NUM_TURNS - 1)
        self.assertGreaterEqual(
            prefix_count, expected, f"Expected at least {expected} prefix pairs"
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestBenchServingFunctionality` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBenchServingFunctionality` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 108-108: Define class TestBenchServingCustomHeaders
```python
class TestBenchServingCustomHeaders(CustomTestCase):
```
**EN:** This declaration introduces the `TestBenchServingCustomHeaders` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBenchServingCustomHeaders` 测试类，并说明它通过继承承担的职责。

### Lines 109-117: Run test: parse custom headers
```python
    def test_parse_custom_headers(self):
        headers = parse_custom_headers(["MyHeader=MY_VALUE", "Another=value=hello"])
        self.assertEqual(headers, {"MyHeader": "MY_VALUE", "Another": "value=hello"})

        headers = parse_custom_headers(["InvalidNoEquals"])
        self.assertEqual(headers, {})

        headers = parse_custom_headers(["=NoKey"])
        self.assertEqual(headers, {})
```
**EN:** This test method exercises parse custom headers and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parse custom headers 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 120-177: Run test: custom headers sent to server (part 1)
```python
    def test_custom_headers_sent_to_server(self):
        import queue

        received_requests = queue.Queue()

        class HeaderEchoHandler(BaseHTTPRequestHandler):
            def _handle(self):
                received_requests.put(
                    {
                        "method": self.command,
                        "path": self.path,
                        "headers": dict(self.headers),
                    }
                )
                self.send_response(200)
                self.send_header("Content-Type", "application/json")
                self.end_headers()
                if self.path == "/v1/models":
                    self.wfile.write(json.dumps({"data": [{"id": "gpt2"}]}).encode())
                elif self.path == "/generate":
                    self.wfile.write(
                        json.dumps(
                            {"text": "ok", "meta_info": {"completion_tokens": 1}}
                        ).encode()
                    )
                else:
                    self.wfile.write(json.dumps({}).encode())

            do_GET = do_POST = _handle

        server = HTTPServer(("127.0.0.1", 0), HeaderEchoHandler)
        port = server.server_address[1]
        server_thread = threading.Thread(target=server.serve_forever)
        server_thread.daemon = True
        server_thread.start()

        try:
            args = get_benchmark_args(
                base_url=f"http://127.0.0.1:{port}",
                backend="sglang",
                dataset_name="random",
                tokenizer="gpt2",
                num_prompts=1,
                random_input_len=8,
                random_output_len=8,
                header=["X-Custom-Test=TestValue123", "X-Another=AnotherVal"],
            )
            args.warmup_requests = 0
            args.disable_tqdm = True
            run_benchmark(args)
        except Exception:
            pass
        finally:
            server.shutdown()

        all_reqs = []
        while not received_requests.empty():
            all_reqs.append(received_requests.get_nowait())
```
**EN:** This test method exercises custom headers sent to server and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 custom headers sent to server 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 179-187: Run test: custom headers sent to server (part 2)
```python
        generate_reqs = [r for r in all_reqs if r["path"] == "/generate"]
        self.assertGreater(
            len(generate_reqs),
            0,
            f"No /generate request. All: {[r['path'] for r in all_reqs]}",
        )
        headers = generate_reqs[0]["headers"]
        self.assertEqual(headers.get("X-Custom-Test"), "TestValue123")
        self.assertEqual(headers.get("X-Another"), "AnotherVal")
```
**EN:** This test method exercises custom headers sent to server and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 custom headers sent to server 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 190-191: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.bench_serving`, `sglang.benchmark.utils`, `sglang.srt.constants`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `http.server`, `json`, `pathlib`, `queue`, `tempfile`, `threading`, `time`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.get`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
