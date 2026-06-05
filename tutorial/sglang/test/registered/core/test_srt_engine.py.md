# test_srt_engine.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/core/test_srt_engine.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on core srt engine in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 core srt engine 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Document the module
```python
"""
Usage:
python3 -m unittest test_srt_engine.TestSRTEngine.test_4_sync_async_stream_combination
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 6-23: Import dependencies
```python
import asyncio
import json
import unittest
from types import SimpleNamespace

import torch

import sglang as sgl
from sglang.bench_offline_throughput import BenchArgs, throughput_test
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.few_shot_gsm8k_engine import run_eval
from sglang.test.test_utils import (
    DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    CustomTestCase,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also runs an evaluation workflow to measure model quality and registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会运行评测流程以衡量模型质量，并将该用例注册到 CUDA CI 覆盖范围。

### Lines 25-26: Register CI metadata
```python
register_cuda_ci(est_time=387, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=261, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 29-29: Define class TestSRTEngine
```python
class TestSRTEngine(CustomTestCase):
```
**EN:** This declaration introduces the `TestSRTEngine` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSRTEngine` 测试类，并说明它通过继承承担的职责。

### Lines 31-50: Run test: 1 engine runtime consistency
```python
    def test_1_engine_runtime_consistency(self):
        prompt = "Today is a sunny day and I like"
        model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST

        sampling_params = {"temperature": 0, "max_new_tokens": 8}

        engine = sgl.Engine(model_path=model_path, random_seed=42)
        out1 = engine.generate(prompt, sampling_params)["text"]
        engine.shutdown()

        runtime = sgl.Runtime(model_path=model_path, random_seed=42)
        out2 = json.loads(runtime.generate(prompt, sampling_params))["text"]
        runtime.shutdown()

        print("==== Answer 1 ====")
        print(out1)

        print("==== Answer 2 ====")
        print(out2)
        self.assertEqual(out1, out2)
```
**EN:** This test method exercises 1 engine runtime consistency and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 1 engine runtime consistency 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 52-64: Run test: 2 engine runtime encode consistency
```python
    def test_2_engine_runtime_encode_consistency(self):
        prompt = "Today is a sunny day and I like"
        model_path = DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST

        engine = sgl.Engine(model_path=model_path, is_embedding=True, random_seed=42)
        out1 = torch.tensor(engine.encode(prompt)["embedding"])
        engine.shutdown()

        runtime = sgl.Runtime(model_path=model_path, is_embedding=True, random_seed=42)
        out2 = torch.tensor(json.loads(runtime.encode(prompt))["embedding"])
        runtime.shutdown()

        self.assertTrue(torch.allclose(out1, out2, atol=1e-5, rtol=1e-3))
```
**EN:** This test method exercises 2 engine runtime encode consistency and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 2 engine runtime encode consistency 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 66-90: Run test: 3 engine token ids consistency
```python
    def test_3_engine_token_ids_consistency(self):
        # just to ensure there is no issue running multiple generate calls
        prompt = "Today is a sunny day and I like"
        model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        sampling_params = {"temperature": 0, "max_new_tokens": 8}

        engine = sgl.Engine(
            model_path=model_path, random_seed=42, disable_radix_cache=True
        )
        out1 = engine.generate(prompt, sampling_params)["text"]

        tokenizer = get_tokenizer(model_path)
        token_ids = tokenizer.encode(prompt)
        out2 = engine.generate(input_ids=token_ids, sampling_params=sampling_params)[
            "text"
        ]

        engine.shutdown()

        print("==== Answer 1 ====")
        print(out1)

        print("==== Answer 2 ====")
        print(out2)
        self.assertEqual(out1, out2)
```
**EN:** This test method exercises 3 engine token ids consistency and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 3 engine token ids consistency 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 92-140: Run test: 4 sync async stream combination
```python
    def test_4_sync_async_stream_combination(self):
        prompt = "AI safety is"
        sampling_params = {"temperature": 0.8, "top_p": 0.95}

        # Create an LLM.
        llm = sgl.Engine(
            model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
        )

        if True:
            # 1. sync + non streaming
            print("\n\n==== 1. sync + non streaming ====")
            output = llm.generate(prompt, sampling_params)
            print(output["text"])

            # 2. sync + streaming
            print("\n\n==== 2. sync + streaming ====")
            output_generator = llm.generate(prompt, sampling_params, stream=True)
            offset = 0
            for output in output_generator:
                print(output["text"][offset:], end="", flush=True)
                offset = len(output["text"])
            print()

        if True:
            loop = asyncio.get_event_loop()
            # 3. async + non_streaming
            print("\n\n==== 3. async + non streaming ====")
            output = loop.run_until_complete(
                llm.async_generate(prompt, sampling_params)
            )
            print(output["text"])

            # 4. async + streaming
            async def async_streaming(engine):
                generator = await engine.async_generate(
                    prompt, sampling_params, stream=True
                )

                offset = 0
                async for output in generator:
                    print(output["text"][offset:], end="", flush=True)
                    offset = len(output["text"])
                print()

            print("\n\n==== 4. async + streaming ====")
            loop.run_until_complete(async_streaming(llm))

        llm.shutdown()
```
**EN:** This test method exercises 4 sync async stream combination and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 4 sync async stream combination 场景，并验证观测到的行为是否符合预期契约。

### Lines 142-152: Run test: 5 gsm8k
```python
    def test_5_gsm8k(self):

        args = SimpleNamespace(
            model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
            local_data_path=None,
            num_shots=5,
            num_questions=1400,
        )

        metrics = run_eval(args)
        self.assertGreater(metrics["accuracy"], 0.33)
```
**EN:** This test method exercises 5 gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 5 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 154-182: Run test: 6 engine cpu offload
```python
    def test_6_engine_cpu_offload(self):
        prompt = "Today is a sunny day and I like"
        model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST

        sampling_params = {"temperature": 0, "max_new_tokens": 8}

        engine = sgl.Engine(
            model_path=model_path,
            random_seed=42,
            max_total_tokens=128,
        )
        out1 = engine.generate(prompt, sampling_params)["text"]
        engine.shutdown()

        engine = sgl.Engine(
            model_path=model_path,
            random_seed=42,
            max_total_tokens=128,
            cpu_offload_gb=3,
        )
        out2 = engine.generate(prompt, sampling_params)["text"]
        engine.shutdown()

        print("==== Answer 1 ====")
        print(out1)

        print("==== Answer 2 ====")
        print(out2)
        self.assertEqual(out1, out2)
```
**EN:** This test method exercises 6 engine cpu offload and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 6 engine cpu offload 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 184-190: Run test: 7 engine offline throughput
```python
    def test_7_engine_offline_throughput(self):
        server_args = ServerArgs(
            model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
        )
        bench_args = BenchArgs(num_prompts=10)
        result = throughput_test(server_args=server_args, bench_args=bench_args)
        self.assertGreater(result["total_throughput"], 3000)
```
**EN:** This test method exercises 7 engine offline throughput and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 7 engine offline throughput 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 192-219: Run test: 8 engine async encode consistency
```python
    def test_8_engine_async_encode_consistency(self):
        prompt = "Today is a sunny day and I like"
        model_path = DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST

        engine = sgl.Engine(
            model_path=model_path,
            is_embedding=True,
            random_seed=42,
            disable_radix_cache=True,
        )

        # Get sync and async embeddings
        out1 = torch.tensor(engine.encode(prompt)["embedding"])
        loop = asyncio.get_event_loop()
        out2 = torch.tensor(
            loop.run_until_complete(engine.async_encode(prompt))["embedding"]
        )

        engine.shutdown()

        print("\n==== Shapes ====")
        print(f"sync shape: {out1.shape}")
        print(f"async shape: {out2.shape}")

        self.assertTrue(
            torch.allclose(out1, out2, atol=1e-5, rtol=1e-3),
            "Sync and async embeddings are not equal within tolerance",
        )
```
**EN:** This test method exercises 8 engine async encode consistency and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 8 engine async encode consistency 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 222-223: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang`, `sglang.bench_offline_throughput`, `sglang.srt.server_args`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k_engine`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `asyncio`, `json`, `torch`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`
