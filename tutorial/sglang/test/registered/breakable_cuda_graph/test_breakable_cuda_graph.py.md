# test_breakable_cuda_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/breakable_cuda_graph/test_breakable_cuda_graph.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on breakable cuda graph in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 breakable cuda graph 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Document the module
```python
"""Tests for the breakable CUDA graph (BCG) runner.

Two test classes:
- ``TestBreakableCUDAGraphBasic`` / ``TestCopyOutput`` / ``TestBreakGraphHelper``:
  unit tests for the core capture / replay mechanism (simple tensor ops).
- ``TestBreakableCudaGraph``: integration test — spin up Qwen3-8B with
  ``--enable-breakable-cuda-graph`` and check mgsm_en accuracy.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 10-23: Import dependencies
```python
import unittest

import torch

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    SimpleNamespace,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 26-26: Register CI metadata
```python
register_cuda_ci(est_time=79, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 29-32: Define helper: skip if no cuda
```python
def _skip_if_no_cuda(test_func):
    return unittest.skipUnless(torch.cuda.is_available(), "CUDA not available")(
        test_func
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 35-41: Define helper: skip if no cuda bindings
```python
def _skip_if_no_cuda_bindings(test_func):
    try:
        from cuda.bindings import runtime as rt  # noqa: F401

        return test_func
    except ImportError:
        return unittest.skip("cuda-python not installed")(test_func)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 44-44: Define class TestBreakableCUDAGraphBasic
```python
class TestBreakableCUDAGraphBasic(CustomTestCase):
```
**EN:** This declaration introduces the `TestBreakableCUDAGraphBasic` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBreakableCUDAGraphBasic` 测试类，并说明它通过继承承担的职责。

### Lines 45-45: Document the class `TestBreakableCUDAGraphBasic`
```python
    """Test basic breakable CUDA graph capture and replay."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestBreakableCUDAGraphBasic`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestBreakableCUDAGraphBasic`的设计意图。

### Lines 47-65: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA not available")
        try:
            from cuda.bindings import runtime  # noqa: F401
        except ImportError:
            raise unittest.SkipTest("cuda-python not installed")

        from sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph import (
            BreakableCUDAGraph,
            BreakableCUDAGraphCapture,
            eager_on_graph,
        )

        cls.BreakableCUDAGraph = BreakableCUDAGraph
        cls.BreakableCUDAGraphCapture = BreakableCUDAGraphCapture
        cls.eager_on_graph = staticmethod(eager_on_graph)
        cls.device = torch.device("cuda:0")
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 67-81: Run test: no break capture replay
```python
    def test_no_break_capture_replay(self):
        """Capture and replay without any graph breaks should work like normal CUDA graph."""
        x = torch.zeros(4, device=self.device)
        y = torch.zeros(4, device=self.device)

        graph = self.BreakableCUDAGraph()
        stream = torch.cuda.Stream(self.device)
        with self.BreakableCUDAGraphCapture(graph, stream=stream):
            y.copy_(x + 1.0)

        # Replay with new input
        x.fill_(5.0)
        graph.replay()
        torch.cuda.synchronize()
        self.assertTrue(torch.allclose(y, torch.full((4,), 6.0, device=self.device)))
```
**EN:** This test method exercises no break capture replay and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no break capture replay 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 83-105: Run test: single break
```python
    def test_single_break(self):
        """A single graph break should split capture into two segments."""
        x = torch.zeros(4, device=self.device)
        intermediate = torch.zeros(4, device=self.device)
        y = torch.zeros(4, device=self.device)

        @self.eager_on_graph(enable=True)
        def eager_op(src):
            return src * 2.0

        graph = self.BreakableCUDAGraph()
        stream = torch.cuda.Stream(self.device)
        with self.BreakableCUDAGraphCapture(graph, stream=stream):
            intermediate.copy_(x + 1.0)
            broken = eager_op(intermediate)
            y.copy_(broken + 3.0)

        # Replay with new input
        x.fill_(10.0)
        graph.replay()
        torch.cuda.synchronize()
        # x=10 -> intermediate=11 -> eager: 11*2=22 -> y=22+3=25
        self.assertTrue(torch.allclose(y, torch.full((4,), 25.0, device=self.device)))
```
**EN:** This test method exercises single break and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single break 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 107-133: Run test: multiple breaks
```python
    def test_multiple_breaks(self):
        """Multiple graph breaks should produce correct chained results."""
        x = torch.zeros(4, device=self.device)
        y = torch.zeros(4, device=self.device)

        @self.eager_on_graph(enable=True)
        def add_one(src):
            return src + 1.0

        @self.eager_on_graph(enable=True)
        def double(src):
            return src * 2.0

        graph = self.BreakableCUDAGraph()
        stream = torch.cuda.Stream(self.device)
        with self.BreakableCUDAGraphCapture(graph, stream=stream):
            t1 = x + 1.0  # graph segment 1
            t2 = add_one(t1)  # break 1: eager
            t3 = t2 + 1.0  # graph segment 2
            t4 = double(t3)  # break 2: eager
            y.copy_(t4)  # graph segment 3

        # Replay: x=5 -> +1=6 -> add_one=7 -> +1=8 -> double=16
        x.fill_(5.0)
        graph.replay()
        torch.cuda.synchronize()
        self.assertTrue(torch.allclose(y, torch.full((4,), 16.0, device=self.device)))
```
**EN:** This test method exercises multiple breaks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple breaks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 135-147: Run test: eager on graph disabled
```python
    def test_eager_on_graph_disabled(self):
        """@eager_on_graph(enable=False) should be a no-op passthrough."""

        @self.eager_on_graph(enable=False)
        def my_fn(x):
            return x + 1.0

        # Should just be the original function
        t = torch.tensor([1.0, 2.0], device=self.device)
        result = my_fn(t)
        self.assertTrue(
            torch.allclose(result, torch.tensor([2.0, 3.0], device=self.device))
        )
```
**EN:** This test method exercises eager on graph disabled and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 eager on graph disabled 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 149-160: Run test: eager on graph outside capture
```python
    def test_eager_on_graph_outside_capture(self):
        """@eager_on_graph called outside capture should run the function directly."""

        @self.eager_on_graph(enable=True)
        def my_fn(x):
            return x + 1.0

        t = torch.tensor([1.0, 2.0], device=self.device)
        result = my_fn(t)
        self.assertTrue(
            torch.allclose(result, torch.tensor([2.0, 3.0], device=self.device))
        )
```
**EN:** This test method exercises eager on graph outside capture and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 eager on graph outside capture 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 162-187: Run test: replay updates output
```python
    def test_replay_updates_output(self):
        """Replay should produce different results when input buffers change."""
        x = torch.zeros(4, device=self.device)
        y = torch.zeros(4, device=self.device)

        @self.eager_on_graph(enable=True)
        def scale(src):
            return src * 3.0

        graph = self.BreakableCUDAGraph()
        stream = torch.cuda.Stream(self.device)
        with self.BreakableCUDAGraphCapture(graph, stream=stream):
            t = x + 1.0
            t2 = scale(t)
            y.copy_(t2)

        # First replay: x=0 -> 0+1=1 -> 1*3=3
        graph.replay()
        torch.cuda.synchronize()
        self.assertTrue(torch.allclose(y, torch.full((4,), 3.0, device=self.device)))

        # Second replay: x=10 -> 10+1=11 -> 11*3=33
        x.fill_(10.0)
        graph.replay()
        torch.cuda.synchronize()
        self.assertTrue(torch.allclose(y, torch.full((4,), 33.0, device=self.device)))
```
**EN:** This test method exercises replay updates output and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replay updates output 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 190-190: Define class TestCopyOutput
```python
class TestCopyOutput(CustomTestCase):
```
**EN:** This declaration introduces the `TestCopyOutput` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCopyOutput` 测试类，并说明它通过继承承担的职责。

### Lines 191-191: Document the class `TestCopyOutput`
```python
    """Test the _copy_output helper for structured output writeback."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestCopyOutput`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestCopyOutput`的设计意图。

### Lines 193-207: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA not available")
        try:
            from cuda.bindings import runtime  # noqa: F401
        except ImportError:
            raise unittest.SkipTest("cuda-python not installed")

        from sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph import (
            _copy_output,
        )

        cls._copy_output = staticmethod(_copy_output)
        cls.device = torch.device("cuda:0")
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 209-214: Run test: tensor copy
```python
    def test_tensor_copy(self):
        dst = torch.zeros(4, device=self.device)
        src = torch.ones(4, device=self.device) * 5.0
        result = self._copy_output(dst, src)
        self.assertIs(result, dst)
        self.assertTrue(torch.allclose(dst, src))
```
**EN:** This test method exercises tensor copy and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tensor copy 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 216-230: Run test: dict copy
```python
    def test_dict_copy(self):
        dst = {
            "a": torch.zeros(4, device=self.device),
            "b": torch.zeros(4, device=self.device),
        }
        src = {
            "a": torch.ones(4, device=self.device),
            "b": torch.ones(4, device=self.device) * 2.0,
        }
        result = self._copy_output(dst, src)
        self.assertIs(result, dst)
        self.assertTrue(torch.allclose(dst["a"], torch.ones(4, device=self.device)))
        self.assertTrue(
            torch.allclose(dst["b"], torch.ones(4, device=self.device) * 2.0)
        )
```
**EN:** This test method exercises dict copy and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dict copy 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 232-245: Run test: object copy
```python
    def test_object_copy(self):
        class FakeOutput:
            def __init__(self, t, label):
                self.tensor = t
                self.label = label

        dst = FakeOutput(torch.zeros(4, device=self.device), "old")
        src = FakeOutput(torch.ones(4, device=self.device) * 3.0, "new")
        result = self._copy_output(dst, src)
        self.assertIs(result, dst)
        self.assertTrue(
            torch.allclose(dst.tensor, torch.ones(4, device=self.device) * 3.0)
        )
        self.assertEqual(dst.label, "new")
```
**EN:** This test method exercises object copy and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 object copy 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 247-249: Run test: non tensor fallback
```python
    def test_non_tensor_fallback(self):
        result = self._copy_output(42, 99)
        self.assertEqual(result, 99)
```
**EN:** This test method exercises non tensor fallback and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor fallback 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 252-252: Define class TestBreakGraphHelper
```python
class TestBreakGraphHelper(CustomTestCase):
```
**EN:** This declaration introduces the `TestBreakGraphHelper` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBreakGraphHelper` 测试类，并说明它通过继承承担的职责。

### Lines 253-253: Document the class `TestBreakGraphHelper`
```python
    """Test the break_graph() convenience function."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestBreakGraphHelper`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestBreakGraphHelper`的设计意图。

### Lines 255-273: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA not available")
        try:
            from cuda.bindings import runtime  # noqa: F401
        except ImportError:
            raise unittest.SkipTest("cuda-python not installed")

        from sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph import (
            BreakableCUDAGraph,
            BreakableCUDAGraphCapture,
            break_graph,
        )

        cls.BreakableCUDAGraph = BreakableCUDAGraph
        cls.BreakableCUDAGraphCapture = BreakableCUDAGraphCapture
        cls.break_graph = staticmethod(break_graph)
        cls.device = torch.device("cuda:0")
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 275-291: Run test: break graph inserts segment
```python
    def test_break_graph_inserts_segment(self):
        """break_graph() should insert a graph break even though it does nothing."""
        x = torch.zeros(4, device=self.device)
        y = torch.zeros(4, device=self.device)

        graph = self.BreakableCUDAGraph()
        stream = torch.cuda.Stream(self.device)
        with self.BreakableCUDAGraphCapture(graph, stream=stream):
            t = x + 1.0
            self.break_graph()
            y.copy_(t + 2.0)

        x.fill_(10.0)
        graph.replay()
        torch.cuda.synchronize()
        # x=10 -> +1=11 -> break -> +2=13
        self.assertTrue(torch.allclose(y, torch.full((4,), 13.0, device=self.device)))
```
**EN:** This test method exercises break graph inserts segment and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 break graph inserts segment 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 294-294: Define class TestBreakableCudaGraph
```python
class TestBreakableCudaGraph(CustomTestCase):
```
**EN:** This declaration introduces the `TestBreakableCudaGraph` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBreakableCudaGraph` 测试类，并说明它通过继承承担的职责。

### Lines 295-295: Document the class `TestBreakableCudaGraph`
```python
    """Integration: Qwen3-8B with --enable-breakable-cuda-graph on mgsm_en."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestBreakableCudaGraph`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestBreakableCudaGraph`的设计意图。

### Lines 297-308: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "Qwen/Qwen3-8B"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--enable-breakable-cuda-graph",
            ],
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 310-312: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 314-327: Run test: gsm8k accuracy
```python
    def test_gsm8k_accuracy(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mgsm_en",
            num_examples=1319,
            num_threads=1024,
        )

        metrics = run_eval(args)
        score = metrics["score"]
        print(f"mgsm_en accuracy with breakable CUDA graph: {score:.3f}")

        self.assertGreaterEqual(score, 0.80)
```
**EN:** This test method exercises gsm8k accuracy and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k accuracy 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 330-331: Expose unittest entrypoint
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
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `cuda.bindings`, `torch`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
