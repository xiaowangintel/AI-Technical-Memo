# test_lora_load_from_tensor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_lora_load_from_tensor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora load from tensor behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 lora load from tensor 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import unittest

import torch
from huggingface_hub import snapshot_download
from safetensors.torch import load_file

import sglang as sgl
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `unittest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `unittest`, `torch`。

### Lines 13-22: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=102, stage="extra-a", runner_config="1-gpu-large")
register_amd_ci(est_time=90, suite="stage-b-test-1-gpu-small-amd")

MODEL_PATH = "Qwen/Qwen3-0.6B"
LORA_REPO = "charent/self_cognition_Alice"
TEST_PROMPT = "Hello, my name is"
EXPECTED_OUTPUT = (
    " Alice, and I am a software engineer. I am excited to share my journey"
)
MAX_NEW_TOKENS = 16
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 25-25: class TestLoRALoadFromTensor declaration / 类 TestLoRALoadFromTensor 声明
```python
class TestLoRALoadFromTensor(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 26-46: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.engine = sgl.Engine(
            model_path=MODEL_PATH,
            enable_lora=True,
            max_lora_rank=64,
            lora_target_modules=["all"],
            mem_fraction_static=0.6,
            log_level="error",
        )

        lora_adapter = snapshot_download(
            repo_id=LORA_REPO,
            allow_patterns=["adapter_model.safetensors", "adapter_config.json"],
        )
        # Load tensors and config from downloaded adapter
        cls.lora_tensors = load_file(
            os.path.join(lora_adapter, "adapter_model.safetensors")
        )
        with open(os.path.join(lora_adapter, "adapter_config.json"), "r") as f:
            cls.lora_config_dict = json.load(f)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 48-103: test case lora lru eviction / 测试用例 lora lru eviction
```python
    def test_lora_lru_eviction(self):
        print("[Test]Testing LRU LoRA eviction...")
        MAX_LOADED_LORAS = 8
        print(f"[Test]Max loaded LoRAs: {MAX_LOADED_LORAS}")
        test_engine = sgl.Engine(
            model_path=MODEL_PATH,
            enable_lora=True,
            max_lora_rank=64,
            lora_target_modules=["all"],
            mem_fraction_static=0.6,
            log_level="error",
            max_loaded_loras=MAX_LOADED_LORAS,
        )

        # Load 10 LoRA adapters, max allowed is 8
        # This should trigger LRU eviction when we exceed the limit
        TEST_LORA_COUNT = 10
        for i in range(TEST_LORA_COUNT):
            print(f"[Test]Loading LoRA adapter {i+1}/10: self_cognition_Alice_{i}")
            result = test_engine.load_lora_adapter_from_tensors(
                lora_name=f"self_cognition_Alice_{i}",
                tensors=self.lora_tensors,
                config_dict=self.lora_config_dict,
            )
            self.assertTrue(
                result.success,
                f"Failed to load LoRA adapter {i}: {result.error_message}",
            )
            print(
                f"[Test]Successfully loaded LoRA {i+1}, current loaded adapters: {list(result.loaded_adapters.keys())}"
            )

        EXPECTED_LORA_ADAPTERS = [
            "self_cognition_Alice_2",
            "self_cognition_Alice_3",
            "self_cognition_Alice_4",
            "self_cognition_Alice_5",
            "self_cognition_Alice_6",
            "self_cognition_Alice_7",
            "self_cognition_Alice_8",
            "self_cognition_Alice_9",
        ]
        EXPECTED_LORA_COUNT = 8
        self.assertEqual(
            len(result.loaded_adapters),
            EXPECTED_LORA_COUNT,
            f"Loaded adapters count does not match expected result: {len(result.loaded_adapters)} != {EXPECTED_LORA_COUNT}",
        )
        self.assertEqual(
            list(result.loaded_adapters.keys()),
            EXPECTED_LORA_ADAPTERS,
            f"Loaded adapters do not match expected result: {list(result.loaded_adapters.keys())} != {EXPECTED_LORA_ADAPTERS}",
        )
        print(
            f"[Test]LRU eviction test passed! Final loaded adapters: {len(result.loaded_adapters)}"
        )
```
**EN:** This test exercises `test_lora_lru_eviction` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_lru_eviction`。

### Lines 105-147: test case lora e2e load from tensor params / 测试用例 lora e2e load from tensor params
```python
    def test_lora_e2e_load_from_tensor_params(self):
        print("[Test]Testing LoRA load from tensor params...")

        result = self.engine.load_lora_adapter_from_tensors(
            lora_name="self_cognition_Alice",
            tensors=self.lora_tensors,
            config_dict=self.lora_config_dict,
        )
        self.assertTrue(
            result.success,
            f"Failed to load LoRA from tensors: {result.error_message}",
        )

        output_without_lora = self.engine.generate(
            prompt=[TEST_PROMPT],
            sampling_params={
                "max_new_tokens": MAX_NEW_TOKENS,
                "temperature": 0.0,
            },
        )

        output_lora = self.engine.generate(
            prompt=[TEST_PROMPT],
            sampling_params={
                "max_new_tokens": MAX_NEW_TOKENS,
                "temperature": 0.0,
            },
            lora_path=["self_cognition_Alice"],
        )

        print(f"[Without LoRA] {output_without_lora[0]}")
        print(f"[With LoRA]  {output_lora[0]}")
        self.assertNotEqual(
            output_without_lora[0]["text"][: len(EXPECTED_OUTPUT)],
            EXPECTED_OUTPUT,
            "Output before applying LoRA should not match expected result",
        )

        self.assertEqual(
            output_lora[0]["text"][: len(EXPECTED_OUTPUT)],
            EXPECTED_OUTPUT,
            "Output after applying LoRA does not match expected result",
        )
```
**EN:** This test exercises `test_lora_e2e_load_from_tensor_params` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_e2e_load_from_tensor_params`。

### Lines 149-201: test case lora load unload load from tensor params / 测试用例 lora load unload load from tensor params
```python
    def test_lora_load_unload_load_from_tensor_params(self):
        print("[Test]Testing LoRA load, unload, load from tensor params...")

        # Load LoRA adapter from tensors
        result = self.engine.load_lora_adapter_from_tensors(
            lora_name="self_cognition_Alice_multiple",
            tensors=self.lora_tensors,
            config_dict=self.lora_config_dict,
        )
        self.assertTrue(
            result.success,
            f"Failed to load LoRA from tensors: {result.error_message}",
        )

        # Unload LoRA adapter
        result = self.engine.unload_lora_adapter("self_cognition_Alice_multiple")
        self.assertTrue(
            result.success, f"Failed to unload LoRA: {result.error_message}"
        )
        with self.assertRaises(ValueError) as context:
            output_lora = self.engine.generate(
                prompt=[TEST_PROMPT],
                sampling_params={
                    "max_new_tokens": MAX_NEW_TOKENS,
                    "temperature": 0.0,
                },
                lora_path=["self_cognition_Alice_multiple"],
            )
        # Load LoRA adapter again
        result_again = self.engine.load_lora_adapter_from_tensors(
            lora_name="self_cognition_Alice_multiple",
            tensors=self.lora_tensors,
            config_dict=self.lora_config_dict,
        )
        self.assertTrue(
            result_again.success,
            f"Failed to load LoRA from tensors: {result_again.error_message}",
        )
        output_lora_loaded_again = self.engine.generate(
            prompt=[TEST_PROMPT],
            sampling_params={
                "max_new_tokens": MAX_NEW_TOKENS,
                "temperature": 0.0,
            },
            lora_path=["self_cognition_Alice_multiple"],
        )

        print(f"[With LoRA Loaded again]  {output_lora_loaded_again[0]}")
        self.assertEqual(
            output_lora_loaded_again[0]["text"][: len(EXPECTED_OUTPUT)],
            EXPECTED_OUTPUT,
            "Output after applying LoRA does not match expected result",
        )
```
**EN:** This test exercises `test_lora_load_unload_load_from_tensor_params` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_load_unload_load_from_tensor_params`。

### Lines 203-282: test case lora logp diff with huggingface (part 1/2) / 测试用例 lora logp diff with huggingface（第 1/2 部分）
```python
    def test_lora_logp_diff_with_huggingface(self):
        """
        Test comparing SGLang and HuggingFace LoRA logprobs when loading LoRA from tensors.
        This verifies that loading LoRA adapters from tensors produces consistent logprobs
        with HuggingFace.
        """

        from sglang.test.runners import HFRunner, SRTRunner
        from sglang.test.test_utils import DEFAULT_PORT_FOR_SRT_TEST_RUNNER

        print("[Test]Testing LoRA logprob difference with HuggingFace...")

        lora_name = "self_cognition_Alice_logprob_test"
        prompts = [TEST_PROMPT]

        # Step 1: Run SGLang with LoRA loaded from tensors
        print("[Test]Running SGLang with LoRA from tensors...")
        with SRTRunner(
            MODEL_PATH,
            torch_dtype=torch.float16,
            model_type="generation",
            tp_size=1,
            max_loras_per_batch=1,
            lora_backend="triton",
            disable_cuda_graph=False,
            disable_radix_cache=True,
            port=DEFAULT_PORT_FOR_SRT_TEST_RUNNER,
            mem_fraction_static=0.6,
            enable_lora=True,
            max_lora_rank=64,
            lora_target_modules=[
                "q_proj",
                "k_proj",
                "v_proj",
                "o_proj",
                "gate_proj",
                "up_proj",
                "down_proj",
            ],
        ) as srt_runner:
            result = srt_runner.engine.load_lora_adapter_from_tensors(
                lora_name=lora_name,
                tensors=self.lora_tensors,
                config_dict=self.lora_config_dict,
            )
            self.assertTrue(
                result.success,
                f"Failed to load LoRA from tensors: {result.error_message}",
            )

            # Run inference with loaded LoRA
            srt_outputs = srt_runner.forward(
                prompts,
                max_new_tokens=MAX_NEW_TOKENS,
                lora_paths=[lora_name],
            )

        # Step 2: Run HuggingFace with LoRA
        print("[Test]Running HuggingFace with LoRA...")
        torch.cuda.empty_cache()

        with HFRunner(
            MODEL_PATH,
            torch_dtype=torch.float16,
            model_type="generation",
            patch_model_do_sample_false=True,
        ) as hf_runner:
            hf_outputs = hf_runner.forward(
                prompts,
                max_new_tokens=MAX_NEW_TOKENS,
                lora_paths=[LORA_REPO],
            )

        # Step 3: Compare results
        sglang_text = srt_outputs.output_strs[0]
        hf_text = hf_outputs.output_strs[0]

        print(f"[Text Output]")
        print(f"  SGLang:      {sglang_text}")
        print(f"  HuggingFace: {hf_text}")
```
**EN:** Test comparing SGLang and HuggingFace LoRA logprobs when loading LoRA from tensors. This test exercises `test_lora_logp_diff_with_huggingface` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test comparing SGLang and HuggingFace LoRA logprobs when loading LoRA from tensors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logp_diff_with_huggingface`。 这一段对应同一逻辑块的第 1 部分。

### Lines 283-330: test case lora logp diff with huggingface (part 2/2) / 测试用例 lora logp diff with huggingface（第 2/2 部分）
```python

        # Compare prefill (input) logprobs
        sglang_prefill = torch.tensor(srt_outputs.top_input_logprobs[0])
        hf_prefill = torch.tensor(hf_outputs.top_input_logprobs[0])

        prefill_diff = torch.abs(sglang_prefill - hf_prefill)
        prefill_max_diff = torch.max(prefill_diff).item()
        prefill_mean_diff = torch.mean(prefill_diff).item()

        print(f"\n[Prefill Logprob Comparison]")
        print(f"  Shape:           {list(sglang_prefill.shape)}")
        print(f"  Max difference:  {prefill_max_diff:.6e}")
        print(f"  Mean difference: {prefill_mean_diff:.6e}")

        # Compare decode (output) logprobs
        sglang_decode = torch.tensor(srt_outputs.top_output_logprobs[0])
        hf_decode = torch.tensor(hf_outputs.top_output_logprobs[0])

        decode_diff = torch.abs(sglang_decode - hf_decode)
        decode_max_diff = torch.max(decode_diff).item()
        decode_mean_diff = torch.mean(decode_diff).item()

        print(f"\n[Decode Logprob Comparison]")
        print(f"  Shape:           {list(sglang_decode.shape)}")
        print(f"  Max difference:  {decode_max_diff:.6e}")
        print(f"  Mean difference: {decode_mean_diff:.6e}")

        # Assert logprobs are close (threshold 1e-1)
        LOGPROB_THRESHOLD = 1e-1
        self.assertLess(
            prefill_max_diff,
            LOGPROB_THRESHOLD,
            f"Prefill logprob max difference too large: {prefill_max_diff:.6e} > {LOGPROB_THRESHOLD:.0e}",
        )
        self.assertLess(
            decode_max_diff,
            LOGPROB_THRESHOLD,
            f"Decode logprob max difference too large: {decode_max_diff:.6e} > {LOGPROB_THRESHOLD:.0e}",
        )

        # Verify text outputs match expected
        self.assertEqual(
            sglang_text[: len(EXPECTED_OUTPUT)],
            EXPECTED_OUTPUT,
            "SGLang output does not match expected result",
        )

        print("\n[Test]LoRA logprob comparison test passed!")
```
**EN:** Test comparing SGLang and HuggingFace LoRA logprobs when loading LoRA from tensors. This test exercises `test_lora_logp_diff_with_huggingface` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test comparing SGLang and HuggingFace LoRA logprobs when loading LoRA from tensors. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logp_diff_with_huggingface`。 这一段对应同一逻辑块的第 2 部分。

### Lines 332-362: test case lora e2e load from flattened bucket / 测试用例 lora e2e load from flattened bucket
```python
    def test_lora_e2e_load_from_flattened_bucket(self):
        """Test loading LoRA via FlattenedTensorBucket format (RL weight sync path)."""
        from sglang.srt.utils import MultiprocessingSerializer
        from sglang.srt.weight_sync.tensor_bucket import FlattenedTensorBucket

        named_tensors = list(self.lora_tensors.items())
        bucket = FlattenedTensorBucket(named_tensors=[(n, t) for n, t in named_tensors])
        bucket_dict = {
            "flattened_tensor": bucket.get_flattened_tensor(),
            "metadata": bucket.get_metadata(),
        }
        serialized = MultiprocessingSerializer.serialize(bucket_dict, output_str=True)

        result = self.engine.load_lora_adapter_from_tensors(
            lora_name="self_cognition_Alice_flattened",
            tensors=serialized,
            config_dict=self.lora_config_dict,
            load_format="flattened_bucket",
        )
        self.assertTrue(result.success, f"Failed: {result.error_message}")

        output = self.engine.generate(
            prompt=[TEST_PROMPT],
            sampling_params={"max_new_tokens": MAX_NEW_TOKENS, "temperature": 0.0},
            lora_path=["self_cognition_Alice_flattened"],
        )
        self.assertEqual(
            output[0]["text"][: len(EXPECTED_OUTPUT)],
            EXPECTED_OUTPUT,
            "Output after applying LoRA via flattened bucket does not match expected",
        )
```
**EN:** Test loading LoRA via FlattenedTensorBucket format (RL weight sync path). This test exercises `test_lora_e2e_load_from_flattened_bucket` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test loading LoRA via FlattenedTensorBucket format (RL weight sync path). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_e2e_load_from_flattened_bucket`。

### Lines 364-366: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        cls.engine.shutdown()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 369-370: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestLoRALoadFromTensor`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRALoadFromTensor.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLoRALoadFromTensor.test_lora_lru_eviction`: This test exercises `test_lora_lru_eviction` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_lru_eviction`。
- `TestLoRALoadFromTensor.test_lora_e2e_load_from_tensor_params`: This test exercises `test_lora_e2e_load_from_tensor_params` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_e2e_load_from_tensor_params`。
- `TestLoRALoadFromTensor.test_lora_load_unload_load_from_tensor_params`: This test exercises `test_lora_load_unload_load_from_tensor_params` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_load_unload_load_from_tensor_params`。
- `TestLoRALoadFromTensor.test_lora_logp_diff_with_huggingface`: Test comparing SGLang and HuggingFace LoRA logprobs when loading LoRA from tensors. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logp_diff_with_huggingface`。
- `TestLoRALoadFromTensor.test_lora_e2e_load_from_flattened_bucket`: Test loading LoRA via FlattenedTensorBucket format (RL weight sync path). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_e2e_load_from_flattened_bucket`。
- `TestLoRALoadFromTensor.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `huggingface_hub`, `safetensors.torch`
- **Internal modules / 内部模块**: `sglang`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 370
