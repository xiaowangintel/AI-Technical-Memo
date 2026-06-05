# test_schedule_simulator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/test_schedule_simulator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on debug utils schedule simulator in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 debug utils schedule simulator 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Import dependencies
```python
import json
import subprocess
import sys
import tempfile
import unittest

from sglang.srt.debug_utils.schedule_simulator import (
    AttentionComputeBalancednessRecorder,
    BatchSizeBalancednessRecorder,
    FIFOScheduler,
    GPUState,
    RandomRouter,
    RoundRobinRouter,
    SimRequest,
    SimulationResult,
    Simulator,
    StepRecord,
    StickyRouter,
    create_arg_parser,
    generate_gsp_requests,
    generate_random_requests,
    load_from_request_logger,
    main,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 28-28: Register CI metadata
```python
register_cpu_ci(est_time=120, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 34-34: Define class TestSimRequest
```python
class TestSimRequest(CustomTestCase):
```
**EN:** This declaration introduces the `TestSimRequest` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSimRequest` 测试类，并说明它通过继承承担的职责。

### Lines 35-39: Run test: basic
```python
    def test_basic(self):
        req = SimRequest(request_id="r1", input_len=100, output_len=50)
        self.assertEqual(req.decoded_tokens, 0)
        self.assertEqual(req.seq_len(), 100)
        self.assertFalse(req.is_finished())
```
**EN:** This test method exercises basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 41-45: Run test: seq len with decoded
```python
    def test_seq_len_with_decoded(self):
        req = SimRequest(
            request_id="r1", input_len=100, output_len=50, decoded_tokens=10
        )
        self.assertEqual(req.seq_len(), 110)
```
**EN:** This test method exercises seq len with decoded and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 seq len with decoded 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 47-51: Run test: is finished
```python
    def test_is_finished(self):
        req = SimRequest(
            request_id="r1", input_len=100, output_len=50, decoded_tokens=50
        )
        self.assertTrue(req.is_finished())
```
**EN:** This test method exercises is finished and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 is finished 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 54-54: Define class TestGPUState
```python
class TestGPUState(CustomTestCase):
```
**EN:** This declaration introduces the `TestGPUState` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGPUState` 测试类，并说明它通过继承承担的职责。

### Lines 55-62: Run test: batch size
```python
    def test_batch_size(self):
        gpu = GPUState(gpu_id=0, max_total_tokens=10000)
        self.assertEqual(gpu.batch_size(), 0)
        gpu.running_requests = [
            SimRequest(request_id="r1", input_len=100, output_len=50),
            SimRequest(request_id="r2", input_len=200, output_len=100),
        ]
        self.assertEqual(gpu.batch_size(), 2)
```
**EN:** This test method exercises batch size and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 batch size 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 64-72: Run test: total seq len
```python
    def test_total_seq_len(self):
        gpu = GPUState(gpu_id=0, max_total_tokens=10000)
        gpu.running_requests = [
            SimRequest(request_id="r1", input_len=100, output_len=50),
            SimRequest(
                request_id="r2", input_len=200, output_len=100, decoded_tokens=10
            ),
        ]
        self.assertEqual(gpu.total_seq_len(), 100 + 210)
```
**EN:** This test method exercises total seq len and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 total seq len 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 74-92: Run test: total seq len shared prefix
```python
    def test_total_seq_len_shared_prefix(self):
        gpu = GPUState(gpu_id=0, max_total_tokens=10000)
        gpu.running_requests = [
            SimRequest(
                request_id="r1",
                input_len=150,
                output_len=50,
                group_id="g0",
                prefix_len=100,
            ),
            SimRequest(
                request_id="r2",
                input_len=150,
                output_len=50,
                group_id="g0",
                prefix_len=100,
            ),
        ]
        self.assertEqual(gpu.total_seq_len(), 150 + 50)
```
**EN:** This test method exercises total seq len shared prefix and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 total seq len shared prefix 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 94-114: Run test: total seq len shared prefix with decoded
```python
    def test_total_seq_len_shared_prefix_with_decoded(self):
        gpu = GPUState(gpu_id=0, max_total_tokens=10000)
        gpu.running_requests = [
            SimRequest(
                request_id="r1",
                input_len=150,
                output_len=50,
                decoded_tokens=10,
                group_id="g0",
                prefix_len=100,
            ),
            SimRequest(
                request_id="r2",
                input_len=150,
                output_len=50,
                decoded_tokens=5,
                group_id="g0",
                prefix_len=100,
            ),
        ]
        self.assertEqual(gpu.total_seq_len(), 160 + 55)
```
**EN:** This test method exercises total seq len shared prefix with decoded and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 total seq len shared prefix with decoded 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 116-142: Run test: total seq len multiple groups
```python
    def test_total_seq_len_multiple_groups(self):
        gpu = GPUState(gpu_id=0, max_total_tokens=10000)
        gpu.running_requests = [
            SimRequest(
                request_id="r1",
                input_len=150,
                output_len=50,
                group_id="g0",
                prefix_len=100,
            ),
            SimRequest(
                request_id="r2",
                input_len=150,
                output_len=50,
                group_id="g0",
                prefix_len=100,
            ),
            SimRequest(
                request_id="r3",
                input_len=200,
                output_len=50,
                group_id="g1",
                prefix_len=150,
            ),
            SimRequest(request_id="r4", input_len=80, output_len=20),
        ]
        self.assertEqual(gpu.total_seq_len(), 150 + 50 + 200 + 80)
```
**EN:** This test method exercises total seq len multiple groups and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 total seq len multiple groups 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 145-145: Define class TestRouters
```python
class TestRouters(CustomTestCase):
```
**EN:** This declaration introduces the `TestRouters` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestRouters` 测试类，并说明它通过继承承担的职责。

### Lines 146-150: Run test: round robin
```python
    def test_round_robin(self):
        router = RoundRobinRouter(num_gpus=4)
        req = SimRequest(request_id="r1", input_len=100, output_len=50)
        results = [router.route(req) for _ in range(8)]
        self.assertEqual(results, [0, 1, 2, 3, 0, 1, 2, 3])
```
**EN:** This test method exercises round robin and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 round robin 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 152-156: Run test: random router
```python
    def test_random_router(self):
        router = RandomRouter(num_gpus=4)
        req = SimRequest(request_id="r1", input_len=100, output_len=50)
        results = [router.route(req) for _ in range(100)]
        self.assertTrue(all(0 <= r < 4 for r in results))
```
**EN:** This test method exercises random router and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 random router 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 158-165: Run test: sticky router same group same gpu
```python
    def test_sticky_router_same_group_same_gpu(self):
        router = StickyRouter(num_gpus=4)
        reqs = [
            SimRequest(request_id=f"r{i}", input_len=100, output_len=50, group_id="g0")
            for i in range(10)
        ]
        results = [router.route(req) for req in reqs]
        self.assertEqual(len(set(results)), 1)
```
**EN:** This test method exercises sticky router same group same gpu and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sticky router same group same gpu 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 167-174: Run test: sticky router no group fallback
```python
    def test_sticky_router_no_group_fallback(self):
        router = StickyRouter(num_gpus=4)
        reqs = [
            SimRequest(request_id=f"r{i}", input_len=100, output_len=50)
            for i in range(100)
        ]
        results = [router.route(req) for req in reqs]
        self.assertTrue(all(0 <= r < 4 for r in results))
```
**EN:** This test method exercises sticky router no group fallback and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sticky router no group fallback 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 176-189: Run test: sticky router multiple groups
```python
    def test_sticky_router_multiple_groups(self):
        router = StickyRouter(num_gpus=4)
        for group_id in ["g0", "g1", "g2"]:
            reqs = [
                SimRequest(
                    request_id=f"{group_id}_r{i}",
                    input_len=100,
                    output_len=50,
                    group_id=group_id,
                )
                for i in range(5)
            ]
            results = [router.route(req) for req in reqs]
            self.assertEqual(len(set(results)), 1)
```
**EN:** This test method exercises sticky router multiple groups and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sticky router multiple groups 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 192-192: Define class TestFIFOScheduler
```python
class TestFIFOScheduler(CustomTestCase):
```
**EN:** This declaration introduces the `TestFIFOScheduler` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFIFOScheduler` 测试类，并说明它通过继承承担的职责。

### Lines 193-202: Run test: runs pending requests
```python
    def test_runs_pending_requests(self):
        scheduler = FIFOScheduler()
        gpu = GPUState(gpu_id=0, max_total_tokens=10000)
        gpu.pending_requests = [
            SimRequest(request_id=f"r{i}", input_len=100, output_len=50)
            for i in range(3)
        ]
        scheduler.schedule(gpu)
        self.assertEqual(len(gpu.running_requests), 3)
        self.assertEqual(len(gpu.pending_requests), 0)
```
**EN:** This test method exercises runs pending requests and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 runs pending requests 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 204-213: Run test: respects token limit
```python
    def test_respects_token_limit(self):
        scheduler = FIFOScheduler()
        gpu = GPUState(gpu_id=0, max_total_tokens=250)
        gpu.pending_requests = [
            SimRequest(request_id=f"r{i}", input_len=100, output_len=50)
            for i in range(5)
        ]
        scheduler.schedule(gpu)
        self.assertEqual(len(gpu.running_requests), 2)
        self.assertEqual(len(gpu.pending_requests), 3)
```
**EN:** This test method exercises respects token limit and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 respects token limit 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 215-225: Run test: evicts lifo when over budget
```python
    def test_evicts_lifo_when_over_budget(self):
        scheduler = FIFOScheduler()
        gpu = GPUState(gpu_id=0, max_total_tokens=250)
        gpu.running_requests = [
            SimRequest(request_id=f"r{i}", input_len=100, output_len=50)
            for i in range(3)
        ]  # 300 tokens total
        scheduler.schedule(gpu)
        self.assertEqual(len(gpu.running_requests), 2)
        self.assertEqual(len(gpu.pending_requests), 1)
        self.assertEqual(gpu.pending_requests[0].request_id, "r2")
```
**EN:** This test method exercises evicts lifo when over budget and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 evicts lifo when over budget 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 228-228: Define class TestMetrics
```python
class TestMetrics(CustomTestCase):
```
**EN:** This declaration introduces the `TestMetrics` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMetrics` 测试类，并说明它通过继承承担的职责。

### Lines 229-242: Run test: batch size balancedness
```python
    def test_batch_size_balancedness(self):
        recorder = BatchSizeBalancednessRecorder()
        gpu_states = [GPUState(gpu_id=i, max_total_tokens=10000) for i in range(2)]
        gpu_states[0].running_requests = [
            SimRequest(request_id="r1", input_len=100, output_len=50)
        ]
        gpu_states[1].running_requests = [
            SimRequest(request_id="r2", input_len=100, output_len=50),
            SimRequest(request_id="r3", input_len=100, output_len=50),
        ]
        recorder.on_step_end(0, gpu_states)
        self.assertAlmostEqual(
            recorder.get_summary()["batch_size_balancedness_mean"], 0.75
        )
```
**EN:** This test method exercises batch size balancedness and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 batch size balancedness 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 244-256: Run test: attention compute balancedness
```python
    def test_attention_compute_balancedness(self):
        recorder = AttentionComputeBalancednessRecorder()
        gpu_states = [GPUState(gpu_id=i, max_total_tokens=10000) for i in range(2)]
        gpu_states[0].running_requests = [
            SimRequest(request_id="r1", input_len=100, output_len=50)
        ]
        gpu_states[1].running_requests = [
            SimRequest(request_id="r2", input_len=200, output_len=50)
        ]
        recorder.on_step_end(0, gpu_states)
        self.assertAlmostEqual(
            recorder.get_summary()["attention_compute_balancedness_mean"], 0.75
        )
```
**EN:** This test method exercises attention compute balancedness and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 attention compute balancedness 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 258-260: Run test: empty history
```python
    def test_empty_history(self):
        recorder = BatchSizeBalancednessRecorder()
        self.assertEqual(recorder.get_summary()["batch_size_balancedness_mean"], 0.0)
```
**EN:** This test method exercises empty history and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty history 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 262-268: Run test: all zero batch size
```python
    def test_all_zero_batch_size(self):
        recorder = BatchSizeBalancednessRecorder()
        gpu_states = [GPUState(gpu_id=i, max_total_tokens=10000) for i in range(2)]
        recorder.on_step_end(0, gpu_states)
        self.assertAlmostEqual(
            recorder.get_summary()["batch_size_balancedness_mean"], 1.0
        )
```
**EN:** This test method exercises all zero batch size and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all zero batch size 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 271-271: Define class TestDataLoader
```python
class TestDataLoader(CustomTestCase):
```
**EN:** This declaration introduces the `TestDataLoader` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDataLoader` 测试类，并说明它通过继承承担的职责。

### Lines 272-295: Run test: load from request logger
```python
    def test_load_from_request_logger(self):
        log_data = [
            {"event": "request.received", "rid": "r1", "obj": {"text": "hello"}},
            {
                "event": "request.finished",
                "rid": "r1",
                "out": {"meta_info": {"prompt_tokens": 100, "completion_tokens": 50}},
            },
            {
                "event": "request.finished",
                "rid": "r2",
                "out": {"meta_info": {"prompt_tokens": 200, "completion_tokens": 100}},
            },
        ]
        with tempfile.NamedTemporaryFile(mode="w", suffix=".log", delete=False) as f:
            for item in log_data:
                f.write(json.dumps(item) + "\n")
            f.flush()
            requests = load_from_request_logger(f.name)

        self.assertEqual(len(requests), 2)
        self.assertEqual(requests[0].request_id, "r1")
        self.assertEqual(requests[0].input_len, 100)
        self.assertEqual(requests[1].input_len, 200)
```
**EN:** This test method exercises load from request logger and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 load from request logger 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 297-301: Run test: empty file
```python
    def test_empty_file(self):
        with tempfile.NamedTemporaryFile(mode="w", suffix=".log", delete=False) as f:
            f.write("")
            f.flush()
            self.assertEqual(len(load_from_request_logger(f.name)), 0)
```
**EN:** This test method exercises empty file and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty file 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 304-304: Define class TestDataSynthesis
```python
class TestDataSynthesis(CustomTestCase):
```
**EN:** This declaration introduces the `TestDataSynthesis` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDataSynthesis` 测试类，并说明它通过继承承担的职责。

### Lines 305-312: Run test: generate basic
```python
    def test_generate_basic(self):
        requests = generate_random_requests(
            num_requests=10, input_len=100, output_len=50
        )
        self.assertEqual(len(requests), 10)
        for req in requests:
            self.assertEqual(req.input_len, 100)
            self.assertEqual(req.output_len, 50)
```
**EN:** This test method exercises generate basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 314-320: Run test: generate with range ratio
```python
    def test_generate_with_range_ratio(self):
        requests = generate_random_requests(
            num_requests=100, input_len=100, output_len=50, range_ratio=0.5, seed=42
        )
        for req in requests:
            self.assertGreaterEqual(req.input_len, 50)
            self.assertLessEqual(req.input_len, 100)
```
**EN:** This test method exercises generate with range ratio and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate with range ratio 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 322-330: Run test: generate with seed
```python
    def test_generate_with_seed(self):
        r1 = generate_random_requests(
            num_requests=10, input_len=100, output_len=50, range_ratio=0.5, seed=42
        )
        r2 = generate_random_requests(
            num_requests=10, input_len=100, output_len=50, range_ratio=0.5, seed=42
        )
        for a, b in zip(r1, r2):
            self.assertEqual(a.input_len, b.input_len)
```
**EN:** This test method exercises generate with seed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate with seed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 332-346: Run test: generate gsp basic
```python
    def test_generate_gsp_basic(self):
        requests = generate_gsp_requests(
            num_groups=4,
            prompts_per_group=3,
            system_prompt_len=100,
            question_len=50,
            output_len=25,
            seed=42,
        )
        self.assertEqual(len(requests), 12)
        for req in requests:
            self.assertIsNotNone(req.group_id)
            self.assertEqual(req.prefix_len, 100)
            self.assertEqual(req.input_len, 150)
            self.assertEqual(req.output_len, 25)
```
**EN:** This test method exercises generate gsp basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate gsp basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 348-362: Run test: generate gsp group assignment
```python
    def test_generate_gsp_group_assignment(self):
        requests = generate_gsp_requests(
            num_groups=3,
            prompts_per_group=2,
            system_prompt_len=100,
            question_len=50,
            output_len=25,
            seed=42,
        )
        group_counts = {}
        for req in requests:
            group_counts[req.group_id] = group_counts.get(req.group_id, 0) + 1
        self.assertEqual(len(group_counts), 3)
        for count in group_counts.values():
            self.assertEqual(count, 2)
```
**EN:** This test method exercises generate gsp group assignment and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate gsp group assignment 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 364-378: Run test: generate gsp with range ratio
```python
    def test_generate_gsp_with_range_ratio(self):
        requests = generate_gsp_requests(
            num_groups=4,
            prompts_per_group=5,
            system_prompt_len=100,
            question_len=50,
            output_len=25,
            range_ratio=0.5,
            seed=42,
        )
        for req in requests:
            self.assertGreaterEqual(req.prefix_len, 50)
            self.assertLessEqual(req.prefix_len, 100)
            self.assertGreaterEqual(req.input_len - req.prefix_len, 25)
            self.assertLessEqual(req.input_len - req.prefix_len, 50)
```
**EN:** This test method exercises generate gsp with range ratio and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate gsp with range ratio 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 380-393: Run test: generate gsp shuffled
```python
    def test_generate_gsp_shuffled(self):
        requests = generate_gsp_requests(
            num_groups=4,
            prompts_per_group=10,
            system_prompt_len=100,
            question_len=50,
            output_len=25,
            seed=42,
        )
        group_ids = [req.group_id for req in requests]
        is_sorted = all(
            group_ids[i] <= group_ids[i + 1] for i in range(len(group_ids) - 1)
        )
        self.assertFalse(is_sorted)
```
**EN:** This test method exercises generate gsp shuffled and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 generate gsp shuffled 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 396-396: Define class TestSimulator
```python
class TestSimulator(CustomTestCase):
```
**EN:** This declaration introduces the `TestSimulator` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSimulator` 测试类，并说明它通过继承承担的职责。

### Lines 397-415: Run test: basic run
```python
    def test_basic_run(self):
        requests = [
            SimRequest(request_id=f"r{i}", input_len=10, output_len=5)
            for i in range(10)
        ]
        sim = Simulator(
            num_gpus_per_engine=2,
            router=RoundRobinRouter(num_gpus=2),
            scheduler=FIFOScheduler(),
            recorders=[
                BatchSizeBalancednessRecorder(),
                AttentionComputeBalancednessRecorder(),
            ],
            max_total_tokens=100,
        )
        result = sim.run(requests)
        self.assertIsInstance(result, SimulationResult)
        self.assertIn("batch_size_balancedness_mean", result.summary)
        self.assertGreater(len(result.step_records), 0)
```
**EN:** This test method exercises basic run and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic run 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 417-430: Run test: all requests complete
```python
    def test_all_requests_complete(self):
        requests = [
            SimRequest(request_id=f"r{i}", input_len=10, output_len=3) for i in range(4)
        ]
        sim = Simulator(
            num_gpus_per_engine=2,
            router=RoundRobinRouter(num_gpus=2),
            scheduler=FIFOScheduler(),
            max_total_tokens=10000,
        )
        sim.run(requests)
        for gpu in sim.gpu_states:
            self.assertEqual(len(gpu.pending_requests), 0)
            self.assertEqual(len(gpu.running_requests), 0)
```
**EN:** This test method exercises all requests complete and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all requests complete 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 432-440: Run test: empty requests
```python
    def test_empty_requests(self):
        sim = Simulator(
            num_gpus_per_engine=2,
            router=RoundRobinRouter(num_gpus=2),
            scheduler=FIFOScheduler(),
        )
        result = sim.run([])
        self.assertEqual(result.summary, {})
        self.assertEqual(len(result.step_records), 0)
```
**EN:** This test method exercises empty requests and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty requests 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 442-457: Run test: step records
```python
    def test_step_records(self):
        requests = [
            SimRequest(request_id=f"r{i}", input_len=10, output_len=3) for i in range(4)
        ]
        sim = Simulator(
            num_gpus_per_engine=2,
            router=RoundRobinRouter(num_gpus=2),
            scheduler=FIFOScheduler(),
            max_total_tokens=10000,
        )
        result = sim.run(requests)
        self.assertGreater(len(result.step_records), 0)
        for record in result.step_records:
            self.assertIsInstance(record, StepRecord)
            self.assertIn(record.gpu_id, [0, 1])
        self.assertEqual(len([r for r in result.step_records if r.step == 0]), 2)
```
**EN:** This test method exercises step records and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 step records 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 459-479: Run test: preemption due to token growth
```python
    def test_preemption_due_to_token_growth(self):
        requests = [
            SimRequest(request_id="r0", input_len=50, output_len=10),
            SimRequest(request_id="r1", input_len=50, output_len=10),
        ]
        sim = Simulator(
            num_gpus_per_engine=1,
            router=RoundRobinRouter(num_gpus=1),
            scheduler=FIFOScheduler(),
            max_total_tokens=110,
        )
        result = sim.run(requests)

        found_preemption = False
        for record in result.step_records:
            if record.running_count == 1 and record.pending_count == 1:
                found_preemption = True
                break
        self.assertTrue(
            found_preemption, "Expected preemption to occur due to token growth"
        )
```
**EN:** This test method exercises preemption due to token growth and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 preemption due to token growth 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 485-485: Define class TestCLI
```python
class TestCLI(CustomTestCase):
```
**EN:** This declaration introduces the `TestCLI` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCLI` 测试类，并说明它通过继承承担的职责。

### Lines 486-491: Define helper: run cli
```python
    def _run_cli(self, *args):
        return subprocess.run(
            [sys.executable, "-m", "sglang.srt.debug_utils.schedule_simulator", *args],
            capture_output=True,
            text=True,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestCLI` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestCLI` 内部调用，从而让场景结构更清晰。

### Lines 493-495: Define helper: assert output contains
```python
    def _assert_output_contains(self, output: str, expected_lines: str):
        for line in expected_lines.strip().split("\n"):
            self.assertIn(line, output)
```
**EN:** This helper function encapsulates reusable logic inside `TestCLI` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestCLI` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 497-523: Run test: cli basic
```python
    def test_cli_basic(self):
        log_data = [
            {
                "event": "request.finished",
                "rid": "r1",
                "out": {"meta_info": {"prompt_tokens": 100, "completion_tokens": 50}},
            },
            {
                "event": "request.finished",
                "rid": "r2",
                "out": {"meta_info": {"prompt_tokens": 200, "completion_tokens": 100}},
            },
        ]
        with tempfile.NamedTemporaryFile(mode="w", suffix=".log", delete=False) as f:
            for item in log_data:
                f.write(json.dumps(item) + "\n")
            input_file = f.name
        with tempfile.NamedTemporaryFile(mode="w", suffix=".json", delete=False) as f:
            output_file = f.name

        result = self._run_cli(
            "--input", input_file, "--num-gpus-per-engine", "2", "--output", output_file
        )
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self.assertIn("Loaded 2 requests", result.stdout)
        with open(output_file) as f:
            self.assertIn("batch_size_balancedness_mean", json.load(f))
```
**EN:** This test method exercises cli basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cli basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 525-540: Run test: cli random router
```python
    def test_cli_random_router(self):
        log_data = [
            {
                "event": "request.finished",
                "rid": "r1",
                "out": {"meta_info": {"prompt_tokens": 100, "completion_tokens": 50}},
            }
        ]
        with tempfile.NamedTemporaryFile(mode="w", suffix=".log", delete=False) as f:
            for item in log_data:
                f.write(json.dumps(item) + "\n")
            input_file = f.name

        result = self._run_cli("--input", input_file, "--router", "random")
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self.assertIn("router=random", result.stdout)
```
**EN:** This test method exercises cli random router and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cli random router 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 542-568: Run test: e2e sticky router group locality
```python
    def test_e2e_sticky_router_group_locality(self):
        result = self._run_cli(
            "--synth-gsp",
            "--synth-gsp-num-groups",
            "1",
            "--synth-gsp-prompts-per-group",
            "4",
            "--synth-gsp-system-prompt-len",
            "10",
            "--synth-gsp-question-len",
            "10",
            "--synth-gsp-output-len",
            "2",
            "--synth-seed",
            "42",
            "--num-gpus-per-engine",
            "2",
            "--router",
            "sticky",
            "--max-total-tokens",
            "1000",
            "--log-level",
            "2",
        )
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self.assertIn("R=4:", result.stdout)
        self.assertIn("R=0:-", result.stdout)
```
**EN:** This test method exercises e2e sticky router group locality and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 e2e sticky router group locality 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 570-585: Run test: cli synthetic
```python
    def test_cli_synthetic(self):
        result = self._run_cli(
            "--synthetic",
            "--synth-random-num-requests",
            "100",
            "--synth-random-input-len",
            "512",
            "--synth-random-output-len",
            "128",
            "--synth-random-range-ratio",
            "0.5",
            "--num-gpus-per-engine",
            "4",
        )
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self.assertIn("Generated 100 random requests", result.stdout)
```
**EN:** This test method exercises cli synthetic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cli synthetic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 587-600: Run test: cli log level
```python
    def test_cli_log_level(self):
        result = self._run_cli(
            "--synthetic",
            "--synth-random-num-requests",
            "10",
            "--synth-random-output-len",
            "5",
            "--num-gpus-per-engine",
            "2",
            "--log-level",
            "1",
        )
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self.assertIn("step=", result.stdout)
```
**EN:** This test method exercises cli log level and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cli log level 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 602-630: Run test: e2e simple no queuing
```python
    def test_e2e_simple_no_queuing(self):
        result = self._run_cli(
            "--synthetic",
            "--synth-random-num-requests",
            "4",
            "--synth-random-input-len",
            "10",
            "--synth-random-output-len",
            "2",
            "--synth-random-range-ratio",
            "1.0",
            "--synth-seed",
            "42",
            "--num-gpus-per-engine",
            "2",
            "--max-total-tokens",
            "10000",
            "--log-level",
            "2",
        )
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self.assertIn(
            "step=0    | GPU0[R=2:syn0,syn2 Q=0:-] | GPU1[R=2:syn1,syn3 Q=0:-]",
            result.stdout,
        )
        self.assertIn(
            "step=1    | GPU0[R=0:- Q=0:-] | GPU1[R=0:- Q=0:-]", result.stdout
        )
        self.assertIn("batch_size_balancedness_mean: 1.0000", result.stdout)
```
**EN:** This test method exercises e2e simple no queuing and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 e2e simple no queuing 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 632-662: Run test: e2e queuing due to token limit
```python
    def test_e2e_queuing_due_to_token_limit(self):
        result = self._run_cli(
            "--synthetic",
            "--synth-random-num-requests",
            "4",
            "--synth-random-input-len",
            "100",
            "--synth-random-output-len",
            "3",
            "--synth-random-range-ratio",
            "1.0",
            "--synth-seed",
            "42",
            "--num-gpus-per-engine",
            "1",
            "--max-total-tokens",
            "210",
            "--log-level",
            "2",
        )
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self._assert_output_contains(
            result.stdout,
            """
step=0    | GPU0[R=2:syn0,syn1 Q=2:syn2,syn3]
step=1    | GPU0[R=2:syn0,syn1 Q=2:syn2,syn3]
step=2    | GPU0[R=0:- Q=2:syn2,syn3]
step=3    | GPU0[R=2:syn2,syn3 Q=0:-]
step=4    | GPU0[R=2:syn2,syn3 Q=0:-]
step=5    | GPU0[R=0:- Q=0:-]""",
        )
```
**EN:** This test method exercises e2e queuing due to token limit and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 e2e queuing due to token limit 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 664-694: Run test: e2e retraction due to token growth
```python
    def test_e2e_retraction_due_to_token_growth(self):
        result = self._run_cli(
            "--synthetic",
            "--synth-random-num-requests",
            "2",
            "--synth-random-input-len",
            "50",
            "--synth-random-output-len",
            "10",
            "--synth-random-range-ratio",
            "1.0",
            "--synth-seed",
            "42",
            "--num-gpus-per-engine",
            "1",
            "--max-total-tokens",
            "110",
            "--log-level",
            "2",
        )
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self._assert_output_contains(
            result.stdout,
            """
step=0    | GPU0[R=2:syn0,syn1 Q=0:-]
step=5    | GPU0[R=2:syn0,syn1 Q=0:-]
step=6    | GPU0[R=1:syn0 Q=1:syn1]
step=9    | GPU0[R=0:- Q=1:syn1]
step=10   | GPU0[R=1:syn1 Q=0:-]
step=13   | GPU0[R=0:- Q=0:-]""",
        )
```
**EN:** This test method exercises e2e retraction due to token growth and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 e2e retraction due to token growth 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 696-716: Run test: cli gsp basic
```python
    def test_cli_gsp_basic(self):
        result = self._run_cli(
            "--synth-gsp",
            "--synth-gsp-num-groups",
            "4",
            "--synth-gsp-prompts-per-group",
            "8",
            "--synth-gsp-system-prompt-len",
            "100",
            "--synth-gsp-question-len",
            "50",
            "--synth-gsp-output-len",
            "10",
            "--synth-seed",
            "42",
            "--num-gpus-per-engine",
            "2",
        )
        self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
        self.assertIn("Generated 32 GSP requests", result.stdout)
        self.assertIn("4 groups x 8 prompts", result.stdout)
```
**EN:** This test method exercises cli gsp basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cli gsp basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 718-746: Run test: e2e gsp shared prefix enables batching
```python
    def test_e2e_gsp_shared_prefix_enables_batching(self):
        for has_long_prefix in [True, False]:
            prefix_len, question_len = (50, 10) if has_long_prefix else (10, 50)
            result = self._run_cli(
                "--synth-gsp",
                "--synth-gsp-num-groups",
                "1",
                "--synth-gsp-prompts-per-group",
                "2",
                "--synth-gsp-system-prompt-len",
                str(prefix_len),
                "--synth-gsp-question-len",
                str(question_len),
                "--synth-gsp-output-len",
                "2",
                "--synth-seed",
                "42",
                "--num-gpus-per-engine",
                "1",
                "--max-total-tokens",
                "80",
                "--log-level",
                "2",
            )
            self.assertEqual(result.returncode, 0, f"CLI failed: {result.stderr}")
            if has_long_prefix:
                self.assertIn("R=2:", result.stdout)
            else:
                self.assertNotIn("R=2:", result.stdout)
```
**EN:** This test method exercises e2e gsp shared prefix enables batching and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 e2e gsp shared prefix enables batching 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 749-749: Define class TestLargerScale
```python
class TestLargerScale(CustomTestCase):
```
**EN:** This declaration introduces the `TestLargerScale` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLargerScale` 测试类，并说明它通过继承承担的职责。

### Lines 750-753: Define helper: run main
```python
    def _run_main(self, *cli_args) -> SimulationResult:
        parser = create_arg_parser()
        args = parser.parse_args(cli_args)
        return main(args)
```
**EN:** This helper function encapsulates reusable logic inside `TestLargerScale` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestLargerScale` 内部调用，从而让场景结构更清晰。

### Lines 755-757: Define helper: assert in range
```python
    def _assert_in_range(self, value, lo, hi, name):
        self.assertGreaterEqual(value, lo, f"{name}={value} < {lo}")
        self.assertLessEqual(value, hi, f"{name}={value} > {hi}")
```
**EN:** This helper function encapsulates reusable logic inside `TestLargerScale` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestLargerScale` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 759-789: Run test: vanilla workload random policy
```python
    def test_vanilla_workload_random_policy(self):
        result = self._run_main(
            "--synthetic",
            "--synth-random-num-requests",
            "500000",
            "--synth-random-input-len",
            "32000",
            "--synth-random-output-len",
            "2000",
            "--synth-seed",
            "42",
            "--num-gpus-per-engine",
            "8",
            "--num-engines",
            "250",
            "--router",
            "random",
            "--max-total-tokens",
            "2000000",
            "--stop-criteria",
            "exist_no_pending",
            "--max-steps",
            "1500",
        )
        self._assert_in_range(
            result.summary["attention_compute_balancedness_mean"], 0.95, 1.0, "attn"
        )
        self._assert_in_range(
            result.summary["batch_size_balancedness_mean"], 0.90, 0.98, "bs"
        )
        self._assert_in_range(result.summary["avg_batch_size"], 127, 141, "avg_bs")
```
**EN:** This test method exercises vanilla workload random policy and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 vanilla workload random policy 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 791-818: Define helper: run gsp workload
```python
    def _run_gsp_workload(self, router: str) -> SimulationResult:
        return self._run_main(
            "--synth-gsp",
            "--synth-gsp-num-groups",
            "50000",
            "--synth-gsp-prompts-per-group",
            "100",
            "--synth-gsp-system-prompt-len",
            "31000",
            "--synth-gsp-question-len",
            "1000",
            "--synth-gsp-output-len",
            "8000",
            "--synth-seed",
            "42",
            "--num-gpus-per-engine",
            "8",
            "--num-engines",
            "250",
            "--router",
            router,
            "--max-total-tokens",
            "500000",
            "--stop-criteria",
            "exist_no_pending",
            "--max-steps",
            "1500",
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestLargerScale` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestLargerScale` 内部调用，从而让场景结构更清晰。

### Lines 820-828: Run test: gsp workload random policy
```python
    def test_gsp_workload_random_policy(self):
        result = self._run_gsp_workload("random")
        self._assert_in_range(
            result.summary["attention_compute_balancedness_mean"], 0.90, 0.97, "attn"
        )
        self._assert_in_range(
            result.summary["batch_size_balancedness_mean"], 0.90, 0.97, "bs"
        )
        self._assert_in_range(result.summary["avg_batch_size"], 14, 17, "avg_bs")
```
**EN:** This test method exercises gsp workload random policy and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsp workload random policy 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 830-838: Run test: gsp workload sticky policy
```python
    def test_gsp_workload_sticky_policy(self):
        result = self._run_gsp_workload("sticky")
        self._assert_in_range(
            result.summary["attention_compute_balancedness_mean"], 0.64, 0.71, "attn"
        )
        self._assert_in_range(
            result.summary["batch_size_balancedness_mean"], 0.64, 0.71, "bs"
        )
        self._assert_in_range(result.summary["avg_batch_size"], 31, 36, "avg_bs")
```
**EN:** This test method exercises gsp workload sticky policy and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsp workload sticky policy 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 841-842: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.schedule_simulator`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `json`, `subprocess`, `sys`, `tempfile`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
