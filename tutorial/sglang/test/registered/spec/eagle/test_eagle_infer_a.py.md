# test_eagle_infer_a.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/eagle/test_eagle_infer_a.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates eagle infer a behavior in SGLang's spec / eagle area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 / eagle 领域中与 eagle infer a 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and dependencies / 模块导入与依赖
```python
import random
import unittest

import sglang as sgl
from sglang.srt.environ import envs
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE,
    DEFAULT_DRAFT_MODEL_EAGLE3,
    DEFAULT_TARGET_MODEL_EAGLE,
    DEFAULT_TARGET_MODEL_EAGLE3,
    CustomTestCase,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `random`, `unittest`, `sglang`, `sglang.srt.environ`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `random`, `unittest`, `sglang`, `sglang.srt.environ`。

### Lines 16-16: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=357, stage="extra-a", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 19-19: class TestEAGLEEngine declaration / 类 TestEAGLEEngine 声明
```python
class TestEAGLEEngine(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 20-36: class-level constants and configuration for `TestEAGLEEngine` / 类级常量与配置
```python
    BASE_CONFIG = {
        "model_path": DEFAULT_TARGET_MODEL_EAGLE,
        "speculative_draft_model_path": DEFAULT_DRAFT_MODEL_EAGLE,
        "speculative_algorithm": "EAGLE",
        "speculative_num_steps": 5,
        "speculative_eagle_topk": 4,
        "speculative_num_draft_tokens": 8,
        "mem_fraction_static": 0.7,
        "cuda_graph_max_bs": 5,
        "trust_remote_code": True,
    }
    NUM_CONFIGS = 2

    THRESHOLDS = {
        "batch_avg_accept_len": 1.9,
        "accept_len": 3.6,
    }
```
**EN:** This block defines shared names such as `BASE_CONFIG`, `NUM_CONFIGS`, `THRESHOLDS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `BASE_CONFIG`, `NUM_CONFIGS`, `THRESHOLDS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 38-40: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        envs.SGLANG_ENABLE_SPEC_V2.set(False)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 42-44: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        envs.SGLANG_ENABLE_SPEC_V2.clear()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 46-54: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.prompt = "Today is a sunny day and I like"
        self.sampling_params = {"temperature": 0, "max_new_tokens": 8}

        ref_engine = sgl.Engine(
            model_path=self.BASE_CONFIG["model_path"], cuda_graph_max_bs=1
        )
        self.ref_output = ref_engine.generate(self.prompt, self.sampling_params)["text"]
        ref_engine.shutdown()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 56-77: test case correctness / 测试用例 correctness
```python
    def test_correctness(self):
        configs = [
            # Basic config
            self.BASE_CONFIG,
            # Chunked prefill
            {**self.BASE_CONFIG, "chunked_prefill_size": 4},
        ]

        for i, config in enumerate(configs[: self.NUM_CONFIGS]):
            with self.subTest(i=i):
                print(f"{config=}")
                engine = sgl.Engine(**config, log_level="info", decode_log_interval=10)
                try:
                    self._test_single_generation(engine)
                    self._test_first_token_finish(engine)
                    self._test_batch_generation(engine)
                    self._test_eos_token(engine)
                    self._test_acc_length(engine)
                finally:
                    engine.flush_cache()  # check engine alive
                    engine.shutdown()
                print("=" * 100)
```
**EN:** This test exercises `test_correctness` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_correctness`。

### Lines 79-82: method test single generation / 方法 test single generation
```python
    def _test_single_generation(self, engine):
        output = engine.generate(self.prompt, self.sampling_params)["text"]
        print(f"{output=}, {self.ref_output=}")
        self.assertEqual(output, self.ref_output)
```
**EN:** This block implements `_test_single_generation` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_single_generation`，承担模块行为中的一个聚焦逻辑片段。

### Lines 84-107: method test batch generation / 方法 test batch generation
```python
    def _test_batch_generation(self, engine):
        prompts = [
            "Hello, my name is",
            "The president of the United States is",
            "The capital of France is",
            "The future of AI is",
        ]
        params = {"temperature": 0, "max_new_tokens": 50}

        outputs = engine.generate(prompts, params)
        for prompt, output in zip(prompts, outputs):
            print(f"Prompt: {prompt}")
            print(f"Generated: {output['text']}")
            print("-" * 40)

        print(f"{engine.get_server_info()=}")

        avg_spec_accept_length = engine.get_server_info()["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")
        self.assertGreater(
            avg_spec_accept_length, self.THRESHOLDS["batch_avg_accept_len"]
        )
```
**EN:** This block implements `_test_batch_generation` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_batch_generation`，承担模块行为中的一个聚焦逻辑片段。

### Lines 109-121: method test first token finish / 方法 test first token finish
```python
    def _test_first_token_finish(self, engine):
        prompt = [
            f"There are {i} apples on the table. How to divide them equally?"
            for i in range(8)
        ]
        params = [
            {"temperature": 0, "max_new_tokens": random.randint(1, 3)} for _ in range(8)
        ]
        outputs = engine.generate(prompt, params)
        for i, output in enumerate(outputs):
            print(f"Prompt: {prompt[i]}")
            print(f"Generated: {output['text']}")
            print("-" * 40)
```
**EN:** This block implements `_test_first_token_finish` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_first_token_finish`，承担模块行为中的一个聚焦逻辑片段。

### Lines 123-136: method test eos token / 方法 test eos token
```python
    def _test_eos_token(self, engine):
        prompt = "[INST] <<SYS>>\nYou are a helpful assistant.\n<</SYS>>\nToday is a sunny day and I like [/INST]"
        params = {
            "temperature": 0.1,
            "max_new_tokens": 1024,
            "skip_special_tokens": False,
        }

        tokenizer = get_tokenizer(DEFAULT_TARGET_MODEL_EAGLE)
        output = engine.generate(prompt, params)["text"]
        print(f"{output=}")

        tokens = tokenizer.encode(output, truncation=False)
        self.assertNotIn(tokenizer.eos_token_id, tokens)
```
**EN:** This block implements `_test_eos_token` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_eos_token`，承担模块行为中的一个聚焦逻辑片段。

### Lines 138-160: method test acc length / 方法 test acc length
```python
    def _test_acc_length(self, engine):
        prompt = [
            "Human: Give me a fully functional FastAPI server. Show the python code.\n\nAssistant:",
        ] * 5  # test batched generation
        sampling_params = {"temperature": 0, "max_new_tokens": 512}
        output = engine.generate(prompt, sampling_params)
        output = output[0]

        if "spec_verify_ct" in output["meta_info"]:
            acc_length = (
                output["meta_info"]["completion_tokens"]
                / output["meta_info"]["spec_verify_ct"]
            )
        else:
            acc_length = 1.0

        speed = (
            output["meta_info"]["completion_tokens"]
            / output["meta_info"]["e2e_latency"]
        )
        print(f"{acc_length=:.4f}, {speed=}")

        self.assertGreater(acc_length, self.THRESHOLDS["accept_len"])
```
**EN:** This block implements `_test_acc_length` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_acc_length`，承担模块行为中的一个聚焦逻辑片段。

### Lines 163-163: class TestEAGLEEngineTokenMap declaration / 类 TestEAGLEEngineTokenMap 声明
```python
class TestEAGLEEngineTokenMap(TestEAGLEEngine):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestEAGLEEngine`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestEAGLEEngine`。

### Lines 164-180: class-level constants and configuration for `TestEAGLEEngineTokenMap` / 类级常量与配置
```python
    BASE_CONFIG = {
        "model_path": "meta-llama/Meta-Llama-3-8B-Instruct",
        "speculative_draft_model_path": "lmsys/sglang-EAGLE-LLaMA3-Instruct-8B",
        "speculative_algorithm": "EAGLE",
        "speculative_num_steps": 5,
        "speculative_eagle_topk": 4,
        "speculative_num_draft_tokens": 8,
        "speculative_token_map": "thunlp/LLaMA3-Instruct-8B-FR-Spec/freq_32768.pt",
        "mem_fraction_static": 0.7,
        "cuda_graph_max_bs": 5,
        "dtype": "float16",
    }
    NUM_CONFIGS = 1
    THRESHOLDS = {
        "batch_avg_accept_len": 1.9,
        "accept_len": 2.5,
    }
```
**EN:** This block defines shared names such as `BASE_CONFIG`, `NUM_CONFIGS`, `THRESHOLDS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `BASE_CONFIG`, `NUM_CONFIGS`, `THRESHOLDS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 183-183: class TestEAGLE3Engine declaration / 类 TestEAGLE3Engine 声明
```python
class TestEAGLE3Engine(TestEAGLEEngine):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestEAGLEEngine`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestEAGLEEngine`。

### Lines 184-199: class-level constants and configuration for `TestEAGLE3Engine` / 类级常量与配置
```python
    BASE_CONFIG = {
        "model_path": DEFAULT_TARGET_MODEL_EAGLE3,
        "speculative_draft_model_path": DEFAULT_DRAFT_MODEL_EAGLE3,
        "speculative_algorithm": "EAGLE3",
        "speculative_num_steps": 5,
        "speculative_eagle_topk": 16,
        "speculative_num_draft_tokens": 64,
        "mem_fraction_static": 0.7,
        "cuda_graph_max_bs": 5,
        "dtype": "float16",
    }
    NUM_CONFIGS = 1
    THRESHOLDS = {
        "batch_avg_accept_len": 1.75,
        "accept_len": 3.1,
    }
```
**EN:** This block defines shared names such as `BASE_CONFIG`, `NUM_CONFIGS`, `THRESHOLDS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `BASE_CONFIG`, `NUM_CONFIGS`, `THRESHOLDS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 202-203: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEAGLEEngine`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEEngineTokenMap`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLE3Engine`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEEngine.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEAGLEEngine.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestEAGLEEngine.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEAGLEEngine.test_correctness`: This test exercises `test_correctness` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_correctness`。
- `TestEAGLEEngine._test_single_generation`: This block implements `_test_single_generation` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_single_generation`，承担模块行为中的一个聚焦逻辑片段。
- `TestEAGLEEngine._test_batch_generation`: This block implements `_test_batch_generation` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_batch_generation`，承担模块行为中的一个聚焦逻辑片段。
- `TestEAGLEEngine._test_first_token_finish`: This block implements `_test_first_token_finish` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_first_token_finish`，承担模块行为中的一个聚焦逻辑片段。
- `TestEAGLEEngine._test_eos_token`: This block implements `_test_eos_token` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_eos_token`，承担模块行为中的一个聚焦逻辑片段。
- `TestEAGLEEngine._test_acc_length`: This block implements `_test_acc_length` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_acc_length`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `unittest`
- **Internal modules / 内部模块**: `sglang`, `sglang.srt.environ`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 203
