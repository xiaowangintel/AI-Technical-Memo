# test_release_memory_occupation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_release_memory_occupation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates release memory occupation behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 release memory occupation 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: supporting statements / 辅助语句
```python
"""Test memory release and resume operations for SGLang engine in hybrid RL training.

This test suite evaluates the SGLang engine's memory management capabilities, focusing
on releasing and resuming memory occupation for KV cache and model weights. It simulates
an RL workflow where the SGLang engine acts as a rollout engine for experience collection.
The process involves initializing the engine, sending a small number of requests to simulate
rollout, releasing memory to mimic offloading during RL training, resuming memory occupation,
updating weights with a trained HuggingFace model, and verifying the updated weights.

Detailed in our proposal (https://github.com/sgl-project/sglang/pull/7099), two test cases
are included:

1. Basic Release and Resume: Uses a lower mem_fraction_static (0.6) to control memory allocation
and avoid OOM errors carefully. This test simulates a scenario without multi-stage memory management,
ensuring the engine can release and resume memory occupation while maintaining functionality after
weight updates.

2. Multi-Stage Release and Resume: Employs a higher mem_fraction_static (0.85) to simulate higher
memory pressure, leveraging multi-stage memory management. It sequentially releases and resumes
KV cache and model weights, verifying memory deallocation and reallocation at each stage, and
ensuring correct weight updates and text generation.

3. Tensor Parallel Tests: Tests memory release and resume operations with different tensor parallel
configurations (tp=1, tp=2) to ensure proper memory management in distributed settings. For different
data parallel size, we test it in verl.

NOTE: This test is temporarily disabled.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 30-54: module imports and dependencies / 模块导入与依赖
```python
import os
import time
import unittest

from transformers import AutoModelForCausalLM

import sglang as sgl
from sglang.srt.constants import (
    GPU_MEMORY_TYPE_CUDA_GRAPH,
    GPU_MEMORY_TYPE_KV_CACHE,
    GPU_MEMORY_TYPE_WEIGHTS,
)
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_HYBRID_MAMBA_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST_BASE,
    DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_BASE,
    DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
    CustomTestCase,
    empty_gpu_cache,
    get_gpu_count,
    get_gpu_memory_gb,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `time`, `unittest`, `transformers`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `time`, `unittest`, `transformers`。

### Lines 56-64: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(
    est_time=200,
    stage="base-c",
    runner_config="4-gpu-h100",
    disabled="Temporarily disabled - needs investigation",
)

# (temporarily) set to true to observe memory usage in nvidia-smi more clearly
_DEBUG_EXTRA = False
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 67-67: class TestReleaseMemoryOccupation declaration / 类 TestReleaseMemoryOccupation 声明
```python
class TestReleaseMemoryOccupation(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 68-90: method setup engine / 方法 setup engine
```python
    def _setup_engine(
        self,
        model_name,
        mem_fraction_static=0.8,
        tp_size=1,
        ep_size=1,
        enable_weights_cpu_backup=False,
    ):
        """Common setup for engine and HF model."""

        os.environ["SGLANG_MEMORY_SAVER_CUDA_GRAPH"] = "1"
        engine = sgl.Engine(
            model_path=model_name,
            random_seed=42,
            enable_memory_saver=True,
            mem_fraction_static=mem_fraction_static,
            tp_size=tp_size,
            ep_size=ep_size,
            enable_weights_cpu_backup=enable_weights_cpu_backup,
            # disable_cuda_graph=True,  # for debugging only
        )

        return engine
```
**EN:** Common setup for engine and HF model. This block implements `_setup_engine` and captures one focused piece of the module's behavior.
**CN:** Common setup for engine and HF model. 该代码块实现 `_setup_engine`，承担模块行为中的一个聚焦逻辑片段。

### Lines 92-107: method common test params / 方法 common test params
```python
    def _common_test_params(self):
        """Common test parameters."""
        return {
            "prompt": "Today is a sunny day and I like",
            "sampling_params": {"temperature": 0, "max_new_tokens": 8},
            "expect_output_before_update_weights": " to spend it outdoors. I decided to",
            "expect_output_after_update_weights": " to go for a walk. I like",
            "prompt_moe": "The weather is nice today, and I want to",
            "sampling_params_moe": {"temperature": 0, "max_new_tokens": 16},
            "expect_output_before_update_weights_moe": " go to the park. I have a picnic basket, a book, and a",
            "expect_output_after_update_weights_moe": " go to the park. I have a lot of things to do, but I",
            "prompt_hybrid_mamba": "The weather is nice today, and I want to",
            "sampling_params_hybrid_mamba": {"temperature": 0, "max_new_tokens": 16},
            "expect_output_before_update_weights_hybrid_mamba": " go out for a walk. But I don't know what to wear. Can",
            "expect_output_after_update_weights_hybrid_mamba": " go out for a walk. But I don't know what to wear. Can",
        }
```
**EN:** Common test parameters. This block implements `_common_test_params` and captures one focused piece of the module's behavior.
**CN:** Common test parameters. 该代码块实现 `_common_test_params`，承担模块行为中的一个聚焦逻辑片段。

### Lines 109-118: method test initial generation / 方法 test initial generation
```python
    def _test_initial_generation(
        self, engine, prompt, sampling_params, expect_output_before_update_weights
    ):
        """Test initial generation and memory allocation."""
        print("generate (#1)")
        outputs = engine.generate(prompt, sampling_params)["text"]
        self.assertEqual(outputs, expect_output_before_update_weights)

        if _DEBUG_EXTRA:
            time.sleep(3)
```
**EN:** Test initial generation and memory allocation. This block implements `_test_initial_generation` and captures one focused piece of the module's behavior.
**CN:** Test initial generation and memory allocation. 该代码块实现 `_test_initial_generation`，承担模块行为中的一个聚焦逻辑片段。

### Lines 120-179: test case release and resume occupation / 测试用例 release and resume occupation
```python
    def test_release_and_resume_occupation(self):
        # Without multi-stage release and resume, we need to carefully control the memory fraction to avoid OOM
        model_name = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        assert get_gpu_count() >= 2, "Need at least 2 GPUs for tensor parallel tests"

        for tp_size in [1, 2]:

            print(f"Testing tp_size={tp_size} for test_release_and_resume_occupation")
            engine = self._setup_engine(
                model_name=model_name, mem_fraction_static=0.6, tp_size=tp_size
            )
            params = self._common_test_params()

            self._test_initial_generation(
                engine,
                params["prompt"],
                params["sampling_params"],
                params["expect_output_before_update_weights"],
            )

            t = time.perf_counter()
            gpu_memory_usage_before_release = get_gpu_memory_gb()
            engine.release_memory_occupation()
            gpu_memory_usage_after_release = get_gpu_memory_gb()

            self.assertLess(
                gpu_memory_usage_after_release,
                gpu_memory_usage_before_release,
            )

            print(
                f"Release took {time.perf_counter() - t:.2f}s, memory: {gpu_memory_usage_before_release:.1f} GB → {gpu_memory_usage_after_release:.1f} GB"
            )

            if _DEBUG_EXTRA:
                time.sleep(3)

            t = time.perf_counter()
            engine.resume_memory_occupation()
            print(
                f"Resume took {time.perf_counter() - t:.2f}s, memory: {get_gpu_memory_gb():.1f} GB"
            )

            hf_model_new = AutoModelForCausalLM.from_pretrained(
                DEFAULT_SMALL_MODEL_NAME_FOR_TEST_BASE,
                torch_dtype="bfloat16",
                device_map=get_device(),
            )
            engine.update_weights_from_tensor(list(hf_model_new.named_parameters()))

            # destroy the hf model
            del hf_model_new
            empty_gpu_cache()

            print("generate (#2)")
            outputs = engine.generate(params["prompt"], params["sampling_params"])[
                "text"
            ]
            self.assertEqual(outputs, params["expect_output_after_update_weights"])
            engine.shutdown()
```
**EN:** This test exercises `test_release_and_resume_occupation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_release_and_resume_occupation`。

### Lines 181-226: test case release and resume occupation with weights cpu backup / 测试用例 release and resume occupation with weights cpu backup
```python
    def test_release_and_resume_occupation_with_weights_cpu_backup(self):
        # Test release and resume occupation with weights CPU backup
        model_name = DEFAULT_SMALL_MODEL_NAME_FOR_TEST

        print("Testing test_release_and_resume_occupation_with_weights_cpu_backup")
        engine = self._setup_engine(
            model_name=model_name,
            mem_fraction_static=0.6,
            enable_weights_cpu_backup=True,
        )
        params = self._common_test_params()

        self._test_initial_generation(
            engine,
            params["prompt"],
            params["sampling_params"],
            params["expect_output_before_update_weights"],
        )

        t = time.perf_counter()
        gpu_memory_usage_before_release = get_gpu_memory_gb()
        engine.release_memory_occupation()
        gpu_memory_usage_after_release = get_gpu_memory_gb()

        self.assertLess(
            gpu_memory_usage_after_release,
            gpu_memory_usage_before_release,
        )

        print(
            f"Release took {time.perf_counter() - t:.2f}s, memory: {gpu_memory_usage_before_release:.1f} GB → {gpu_memory_usage_after_release:.1f} GB"
        )

        if _DEBUG_EXTRA:
            time.sleep(3)

        t = time.perf_counter()
        engine.resume_memory_occupation()
        print(
            f"Resume took {time.perf_counter() - t:.2f}s, memory: {get_gpu_memory_gb():.1f} GB"
        )

        print("generate post resume")
        outputs = engine.generate(params["prompt"], params["sampling_params"])["text"]
        self.assertEqual(outputs, params["expect_output_before_update_weights"])
        engine.shutdown()
```
**EN:** This test exercises `test_release_and_resume_occupation_with_weights_cpu_backup` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_release_and_resume_occupation_with_weights_cpu_backup`。

### Lines 228-307: test case multi stage release and resume (part 1/2) / 测试用例 multi stage release and resume（第 1/2 部分）
```python
    def test_multi_stage_release_and_resume(self):
        # With multi-stage release and resume, we can set the memory fraction to 0.85 without concern of OOM
        model_name = DEFAULT_SMALL_MODEL_NAME_FOR_TEST

        for tp_size in [1, 2]:
            if tp_size == 2 and get_gpu_count() < 2:
                continue

            print(f"Testing tp_size={tp_size} for test_multi_stage_release_and_resume")
            os.environ["SGLANG_MEMORY_SAVER_CUDA_GRAPH"] = "1"
            engine = sgl.Engine(
                model_path=model_name,
                random_seed=42,
                enable_memory_saver=True,
                mem_fraction_static=0.85,  # Higher memory pressure
                tp_size=tp_size,
            )
            params = self._common_test_params()

            self._test_initial_generation(
                engine,
                params["prompt"],
                params["sampling_params"],
                params["expect_output_before_update_weights"],
            )

            t = time.perf_counter()
            gpu_memory_usage_before_release = get_gpu_memory_gb()
            engine.release_memory_occupation(tags=[GPU_MEMORY_TYPE_KV_CACHE])

            gpu_memory_usage_after_release_kv_cache = get_gpu_memory_gb()

            self.assertLess(
                gpu_memory_usage_after_release_kv_cache,
                gpu_memory_usage_before_release,
            )

            engine.release_memory_occupation(tags=[GPU_MEMORY_TYPE_WEIGHTS])
            gpu_memory_usage_after_release_weights = get_gpu_memory_gb()

            self.assertLess(
                gpu_memory_usage_after_release_weights,
                gpu_memory_usage_after_release_kv_cache,
            )

            engine.release_memory_occupation(tags=[GPU_MEMORY_TYPE_CUDA_GRAPH])
            gpu_memory_usage_after_release_cuda_graph = get_gpu_memory_gb()

            self.assertLess(
                gpu_memory_usage_after_release_cuda_graph,
                gpu_memory_usage_after_release_weights,
            )

            print(f"Release took {time.perf_counter() - t:.2f}s")
            print(
                f"Memory: {gpu_memory_usage_before_release:.1f} → {gpu_memory_usage_after_release_kv_cache:.1f} → {gpu_memory_usage_after_release_weights:.1f} → {gpu_memory_usage_after_release_cuda_graph:.1f} GB"
            )

            if _DEBUG_EXTRA:
                time.sleep(3)

            t = time.perf_counter()
            gpu_memory_usage_before_resume = get_gpu_memory_gb()

            # gpu_memory_usage_after_release_weights and gpu_memory_usage_before_resume should be close

            self.assertAlmostEqual(
                gpu_memory_usage_after_release_weights,
                gpu_memory_usage_before_resume,
                delta=3.0,
            )
            print(f"Resume weights took {time.perf_counter() - t:.2f}s")

            engine.resume_memory_occupation(tags=[GPU_MEMORY_TYPE_CUDA_GRAPH])
            gpu_memory_usage_after_resume_cuda_graph = get_gpu_memory_gb()

            self.assertGreater(
                gpu_memory_usage_after_resume_cuda_graph,
                gpu_memory_usage_before_resume,
            )
```
**EN:** This test exercises `test_multi_stage_release_and_resume` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_stage_release_and_resume`。 这一段对应同一逻辑块的第 1 部分。

### Lines 308-347: test case multi stage release and resume (part 2/2) / 测试用例 multi stage release and resume（第 2/2 部分）
```python

            engine.resume_memory_occupation(tags=[GPU_MEMORY_TYPE_WEIGHTS])
            gpu_memory_usage_after_resume_weights = get_gpu_memory_gb()

            self.assertGreater(
                gpu_memory_usage_after_resume_weights,
                gpu_memory_usage_after_resume_cuda_graph,
            )

            # Update weights from a trained model to serving engine, and then destroy the trained model
            hf_model_new = AutoModelForCausalLM.from_pretrained(
                DEFAULT_SMALL_MODEL_NAME_FOR_TEST_BASE,
                torch_dtype="bfloat16",
                device_map=get_device(),
            )
            gpu_memory_usage_after_loaded_hf_model = get_gpu_memory_gb()
            engine.update_weights_from_tensor(list(hf_model_new.named_parameters()))

            # destroy the hf model
            del hf_model_new
            empty_gpu_cache()
            engine.resume_memory_occupation(tags=[GPU_MEMORY_TYPE_KV_CACHE])

            gpu_memory_usage_after_resume_kv_cache = get_gpu_memory_gb()
            self.assertGreater(
                gpu_memory_usage_after_resume_kv_cache,
                gpu_memory_usage_after_resume_weights,
            )

            print(f"Resume + update took {time.perf_counter() - t:.2f}s")
            print(
                f"Memory: {gpu_memory_usage_before_resume:.1f} → {gpu_memory_usage_after_resume_cuda_graph:.1f} → {gpu_memory_usage_after_resume_weights:.1f} → {gpu_memory_usage_after_loaded_hf_model:.1f} → {gpu_memory_usage_after_resume_kv_cache:.1f} GB"
            )

            print("generate (#2)")
            outputs = engine.generate(params["prompt"], params["sampling_params"])[
                "text"
            ]
            self.assertEqual(outputs, params["expect_output_after_update_weights"])
            engine.shutdown()
```
**EN:** This test exercises `test_multi_stage_release_and_resume` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_stage_release_and_resume`。 这一段对应同一逻辑块的第 2 部分。

### Lines 349-413: test case moe model release and resume / 测试用例 moe model release and resume
```python
    def test_moe_model_release_and_resume(self):
        # Test with MoE model
        model_name = DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT

        tp_size = ep_size = 2

        print(
            f"Testing tp_size={tp_size} and ep_size={ep_size} for test_moe_model_release_and_resume"
        )
        engine = sgl.Engine(
            model_path=model_name,
            random_seed=42,
            enable_memory_saver=True,
            mem_fraction_static=0.5,
            tp_size=tp_size,
            ep_size=ep_size,
        )
        params = self._common_test_params()

        self._test_initial_generation(
            engine,
            params["prompt_moe"],
            params["sampling_params_moe"],
            params["expect_output_before_update_weights_moe"],
        )

        t = time.perf_counter()
        gpu_memory_usage_before_release = get_gpu_memory_gb()
        engine.release_memory_occupation()
        gpu_memory_usage_after_release = get_gpu_memory_gb()
        self.assertLess(
            gpu_memory_usage_after_release,
            gpu_memory_usage_before_release,
        )

        print(
            f"Release took {time.perf_counter() - t:.2f}s, memory: {gpu_memory_usage_before_release:.1f} GB → {gpu_memory_usage_after_release:.1f} GB"
        )

        if _DEBUG_EXTRA:
            time.sleep(3)

        t = time.perf_counter()
        engine.resume_memory_occupation()
        print(
            f"Resume took {time.perf_counter() - t:.2f}s, memory: {get_gpu_memory_gb():.1f} GB"
        )

        hf_model_new = AutoModelForCausalLM.from_pretrained(
            DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_BASE,
            torch_dtype="bfloat16",
            device_map=get_device(),
        )
        engine.update_weights_from_tensor(list(hf_model_new.named_parameters()))

        # destroy the hf model
        del hf_model_new
        empty_gpu_cache()

        print("generate (#2)")
        outputs = engine.generate(params["prompt_moe"], params["sampling_params_moe"])[
            "text"
        ]
        self.assertEqual(outputs, params["expect_output_after_update_weights_moe"])
        engine.shutdown()
```
**EN:** This test exercises `test_moe_model_release_and_resume` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_model_release_and_resume`。

### Lines 415-473: test case hybrid mamba model release and resume / 测试用例 hybrid mamba model release and resume
```python
    def test_hybrid_mamba_model_release_and_resume(self):
        # Test with Hybrid Mamba model
        model_name = DEFAULT_HYBRID_MAMBA_MODEL_NAME_FOR_TEST

        tp_size = 4

        print(
            f"Testing tp_size={tp_size} for test_hybrid_mamba_model_release_and_resume"
        )
        engine = sgl.Engine(
            model_path=model_name,
            random_seed=42,
            enable_memory_saver=True,
            tp_size=tp_size,
        )
        params = self._common_test_params()

        self._test_initial_generation(
            engine,
            params["prompt_hybrid_mamba"],
            params["sampling_params_hybrid_mamba"],
            params["expect_output_before_update_weights_hybrid_mamba"],
        )

        t = time.perf_counter()
        gpu_memory_usage_before_release = get_gpu_memory_gb()
        engine.release_memory_occupation()
        gpu_memory_usage_after_release = get_gpu_memory_gb()
        self.assertLess(
            gpu_memory_usage_after_release,
            gpu_memory_usage_before_release,
        )

        print(
            f"Release took {time.perf_counter() - t:.2f}s, memory: {gpu_memory_usage_before_release:.1f} GB → {gpu_memory_usage_after_release:.1f} GB"
        )

        if _DEBUG_EXTRA:
            time.sleep(3)

        t = time.perf_counter()
        engine.resume_memory_occupation()
        print(
            f"Resume took {time.perf_counter() - t:.2f}s, memory: {get_gpu_memory_gb():.1f} GB"
        )

        engine.update_weights_from_disk(model_name)

        # destroy the hf model
        empty_gpu_cache()

        print("generate (#2)")
        outputs = engine.generate(
            params["prompt_hybrid_mamba"], params["sampling_params_hybrid_mamba"]
        )["text"]
        self.assertEqual(
            outputs, params["expect_output_after_update_weights_hybrid_mamba"]
        )
        engine.shutdown()
```
**EN:** This test exercises `test_hybrid_mamba_model_release_and_resume` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hybrid_mamba_model_release_and_resume`。

### Lines 476-477: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestReleaseMemoryOccupation`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestReleaseMemoryOccupation._setup_engine`: Common setup for engine and HF model. / 该代码块实现 `_setup_engine`，承担模块行为中的一个聚焦逻辑片段。
- `TestReleaseMemoryOccupation._common_test_params`: Common test parameters. / 该代码块实现 `_common_test_params`，承担模块行为中的一个聚焦逻辑片段。
- `TestReleaseMemoryOccupation._test_initial_generation`: Test initial generation and memory allocation. / 该代码块实现 `_test_initial_generation`，承担模块行为中的一个聚焦逻辑片段。
- `TestReleaseMemoryOccupation.test_release_and_resume_occupation`: This test exercises `test_release_and_resume_occupation` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_release_and_resume_occupation`。
- `TestReleaseMemoryOccupation.test_release_and_resume_occupation_with_weights_cpu_backup`: This test exercises `test_release_and_resume_occupation_with_weights_cpu_backup` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_release_and_resume_occupation_with_weights_cpu_backup`。
- `TestReleaseMemoryOccupation.test_multi_stage_release_and_resume`: This test exercises `test_multi_stage_release_and_resume` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_stage_release_and_resume`。
- `TestReleaseMemoryOccupation.test_moe_model_release_and_resume`: This test exercises `test_moe_model_release_and_resume` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_model_release_and_resume`。
- `TestReleaseMemoryOccupation.test_hybrid_mamba_model_release_and_resume`: This test exercises `test_hybrid_mamba_model_release_and_resume` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hybrid_mamba_model_release_and_resume`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `time`, `unittest`
- **Third-party modules / 第三方模块**: `transformers`
- **Internal modules / 内部模块**: `sglang`, `sglang.srt.constants`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 477
