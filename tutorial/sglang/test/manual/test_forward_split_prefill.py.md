# test_forward_split_prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_forward_split_prefill.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `forward split prefill` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual` 中的 `forward split prefill` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Imports and shared helpers / 导入与共享辅助项
```python
"""
Test forward_split_prefill functionality.

Usage:
python3 -m unittest test_forward_split_prefill.TestForwardSplitPrefill
or
python3 test_forward_split_prefill.py
"""

import unittest

import numpy as np
import torch

from sglang.bench_one_batch import TreeCacheNamespace
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.managers.schedule_batch import Req, ScheduleBatch
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_executor.model_runner import ModelRunner
from sglang.srt.sampling.sampling_params import SamplingParams
from sglang.srt.server_args import PortArgs, ServerArgs
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.utils import get_device
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.test_utils import DEFAULT_SMALL_MODEL_NAME_FOR_TEST, CustomTestCase
```
**EN:** This range imports `unittest`, `numpy`, `torch` and `sglang.bench_one_batch`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 28-31: Class definition for TestForwardSplitPrefill / 类定义
```python
class TestForwardSplitPrefill(CustomTestCase):
    """Test cases for forward_split_prefill functionality."""

    @classmethod
```
**EN:** This range declares `TestForwardSplitPrefill`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 32-52: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        """Set up the test environment once for all tests."""
        cls.model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.tp_size = 1
        cls.device = get_device()

        # Initialize server args
        cls.server_args = ServerArgs(
            model_path=cls.model_path,
            tokenizer_path=cls.model_path,
            host="127.0.0.1",
            disable_cuda_graph=True,  # Disable CUDA graph for testing split prefill
            disable_hybrid_swa_memory=True,
            port=30000,
            tp_size=cls.tp_size,
            mem_fraction_static=0.8,
            trust_remote_code=True,
        )

        cls.port_args = PortArgs.init_new(cls.server_args)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `get_device`, `ServerArgs` and `init_new`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 53-56: Scenario logic / 场景逻辑
```python
        # Load model and tokenizer
        cls.model_config = ModelConfig.from_server_args(cls.server_args)
        cls.model_runner = ModelRunner(
            model_config=cls.model_config,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `from_server_args` and `ModelRunner`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 57-77: Scenario logic / 场景逻辑
```python
            mem_fraction_static=cls.server_args.mem_fraction_static,
            gpu_id=0,
            tp_rank=0,
            tp_size=cls.tp_size,
            pp_rank=0,
            pp_size=1,
            nccl_port=cls.port_args.nccl_port,
            server_args=cls.server_args,
            moe_ep_rank=0,
            moe_ep_size=1,
        )

        cls.tokenizer = get_tokenizer(
            cls.server_args.tokenizer_path,
            tokenizer_mode=cls.server_args.tokenizer_mode,
            trust_remote_code=cls.server_args.trust_remote_code,
        )

        print(
            f"Test with model: {cls.model_path}, num_hidden_layers: {cls.model_config.num_hidden_layers}"
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `get_tokenizer`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 79-102: Helper routines around prepare_test_batch / 辅助例程
```python
    def prepare_test_batch(self, batch_size=2, input_len=128, is_split_prefill=True):
        """Prepare a test batch for split prefill testing."""
        # Create synthetic input
        input_ids = np.random.randint(10, 1000, (batch_size, input_len), dtype=np.int32)

        sampling_params = SamplingParams(
            temperature=0.0,
            max_new_tokens=8,
        )

        reqs = []
        for i in range(batch_size):
            req = Req(
                rid=i,
                origin_input_text="",
                origin_input_ids=list(input_ids[i]),
                sampling_params=sampling_params,
            )
            req.fill_ids = req.origin_input_ids
            req.logprob_start_len = -1
            req.set_extend_input_len(len(req.fill_ids) - len(req.prefix_indices))
            reqs.append(req)

        # Create dummy tree_cache for tests (no prefix caching, just allocation)
```
**EN:** This range implements helper routine(s) `prepare_test_batch` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `randint`, `SamplingParams`, `Req` and `set_extend_input_len`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 103-103: Scenario logic / 场景逻辑
```python
        dummy_tree_cache = TreeCacheNamespace(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `TreeCacheNamespace`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 104-126: Scenario logic / 场景逻辑
```python
            page_size=1,
            device=self.model_runner.device,
            token_to_kv_pool_allocator=self.model_runner.token_to_kv_pool_allocator,
        )

        batch = ScheduleBatch.init_new(
            reqs=reqs,
            req_to_token_pool=self.model_runner.req_to_token_pool,
            token_to_kv_pool_allocator=self.model_runner.token_to_kv_pool_allocator,
            tree_cache=dummy_tree_cache,
            model_config=self.model_config,
            enable_overlap=False,
            spec_algorithm=SpeculativeAlgorithm.NONE,
        )
        if is_split_prefill:
            batch.prepare_for_split_prefill()
        else:
            batch.prepare_for_extend()

        # Create forward batch
        forward_batch = ForwardBatch.init_new(batch, self.model_runner)

        return forward_batch
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `init_new`, `prepare_for_split_prefill` and `prepare_for_extend`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 128-152: Test routines around test_split_prefill_functionality / 测试例程
```python
    def test_split_prefill_functionality(self):
        """Test that split prefill can complete successfully."""
        print("\n=== Testing split prefill functionality ===")

        forward_batch = self.prepare_test_batch(batch_size=2, input_len=64)

        # Reset split index
        forward_batch.split_index = 0

        # Test split prefill in chunks
        num_layers = self.model_config.num_hidden_layers
        chunk_size = max(1, num_layers // 4)  # Split into 4 chunks

        results = []
        split_count = 0

        while forward_batch.split_index < num_layers:
            print(
                f"Processing split {split_count}, split_index: {forward_batch.split_index}"
            )

            result = self.model_runner.forward_split_prefill(
                forward_batch=forward_batch,
                reinit_attn_backend=(split_count == 0),
                forward_count=chunk_size,
```
**EN:** This range defines concrete test routine(s) `test_split_prefill_functionality`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `prepare_test_batch` and `forward_split_prefill`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 153-164: Assertions and result checks / 断言与结果检查
```python
            )

            results.append(result)
            split_count += 1

            # Verify split_index is updated correctly
            expected_next_index = min(split_count * chunk_size, num_layers)
            self.assertEqual(forward_batch.split_index, expected_next_index)

        # The last result should contain logits
        self.assertIsNotNone(results[-1], "Final split should return logits")
        print(f"Split prefill completed in {split_count} splits")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `append`, `assertEqual` and `assertIsNotNone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 166-177: Test routines around test_split_prefill_vs_normal_prefill / 测试例程
```python
    def test_split_prefill_vs_normal_prefill(self):
        """Test that split prefill produces the same results as normal prefill."""
        print("\n=== Testing split prefill vs normal prefill consistency ===")

        forward_batch_normal = self.prepare_test_batch(
            batch_size=2, input_len=128, is_split_prefill=False
        )
        forward_batch_split = self.prepare_test_batch(
            batch_size=2, input_len=128, is_split_prefill=True
        )

        # Ensure same input
```
**EN:** This range defines concrete test routine(s) `test_split_prefill_vs_normal_prefill`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `prepare_test_batch`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 178-190: Scenario logic / 场景逻辑
```python
        forward_batch_split.input_ids = forward_batch_normal.input_ids.clone()
        forward_batch_split.positions = forward_batch_normal.positions.clone()

        # Method 1: Normal extend (prefill)
        print("Running normal extend (prefill)...")
        normal_result = self.model_runner.forward_extend(forward_batch_normal)

        # Method 2: Split prefill
        print("Running split prefill...")
        num_layers = self.model_config.num_hidden_layers
        chunk_size = max(1, num_layers // 3)  # Split into 3 chunks

        split_result = None
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `clone`, `extend` and `forward_extend`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 191-202: Assertions and result checks / 断言与结果检查
```python

        while forward_batch_split.split_index < num_layers:
            result = self.model_runner.forward_split_prefill(
                forward_batch=forward_batch_split,
                forward_count=chunk_size,
            )
            if result is not None:
                split_result = result

        # Compare results
        self.assertIsNotNone(normal_result, "Normal prefill should return result")
        self.assertIsNotNone(split_result, "Split prefill should return result")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `forward_split_prefill` and `assertIsNotNone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 203-215: Assertions and result checks / 断言与结果检查
```python

        # Compare logits shapes
        self.assertEqual(
            normal_result.next_token_logits.shape,
            split_result.next_token_logits.shape,
            "Logits shapes should match",
        )

        # Compare logits values (should be very close due to same computation)
        # Use a larger tolerance for numerical differences in split computation
        torch.testing.assert_close(
            normal_result.next_token_logits,
            split_result.next_token_logits,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `values` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 216-221: Scenario logic / 场景逻辑
```python
            rtol=1e-3,
            atol=1e-3,
            msg="Split prefill and normal prefill should produce similar logits",
        )

        print("✓ Split prefill and normal prefill produce consistent results")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 223-227: Test routines around test_split_prefill_different_chunk_sizes / 测试例程
```python
    def test_split_prefill_different_chunk_sizes(self):
        """Test split prefill with different chunk sizes."""
        print("\n=== Testing split prefill with different chunk sizes ===")

        num_layers = self.model_config.num_hidden_layers
```
**EN:** This range defines concrete test routine(s) `test_split_prefill_different_chunk_sizes`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 228-247: Scenario logic / 场景逻辑
```python
        chunk_sizes = [1, 2, max(1, num_layers // 2), num_layers]

        # Prepare identical batches for each test
        base_batch = self.prepare_test_batch(batch_size=1, input_len=16)
        base_input_ids = base_batch.input_ids.clone()
        base_positions = base_batch.positions.clone()

        results = []

        for chunk_size in chunk_sizes:
            if chunk_size > num_layers:
                continue

            print(f"Testing chunk size: {chunk_size}")

            # Prepare fresh batch
            forward_batch = self.prepare_test_batch(batch_size=1, input_len=16)
            forward_batch.input_ids = base_input_ids.clone()
            forward_batch.positions = base_positions.clone()
            forward_batch.split_index = 0
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `prepare_test_batch` and `clone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 248-252: Scenario logic / 场景逻辑
```python

            # Run split prefill
            split_result = None

            while forward_batch.split_index < num_layers:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 253-272: Assertions and result checks / 断言与结果检查
```python
                result = self.model_runner.forward_split_prefill(
                    forward_batch=forward_batch,
                    forward_count=chunk_size,
                )
                if result is not None:
                    split_result = result

            self.assertIsNotNone(
                split_result,
                f"Split prefill should succeed with chunk_size={chunk_size}",
            )
            results.append(split_result)

        # Compare all results should be identical (same input, same computation)
        if len(results) > 1:
            for i, result in enumerate(results[1:], 1):
                torch.testing.assert_close(
                    results[0].next_token_logits,
                    result.next_token_logits,
                    rtol=1e-3,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `forward_split_prefill`, `assertIsNotNone`, `append` and `identical`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 273-277: Scenario logic / 场景逻辑
```python
                    atol=1e-3,
                    msg=f"Results with different chunk sizes should be identical (chunk_size {chunk_sizes[i]})",
                )

        print("✓ All chunk sizes produce consistent results")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `identical`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 279-302: Test routines around test_split_prefill_edge_cases / 测试例程
```python
    def test_split_prefill_edge_cases(self):
        """Test edge cases for split prefill."""
        print("\n=== Testing split prefill edge cases ===")

        # Test with single layer chunks
        forward_batch = self.prepare_test_batch(batch_size=1, input_len=8)

        # Process one layer at a time
        num_layers = self.model_config.num_hidden_layers
        for layer_idx in range(num_layers):
            result = self.model_runner.forward_split_prefill(
                forward_batch=forward_batch,
                reinit_attn_backend=(layer_idx == 0),
                forward_count=1,  # One layer at a time
            )

            if layer_idx == num_layers - 1:
                # Last layer should return result
                self.assertIsNotNone(result, "Last layer should return logits")
            else:
                # Intermediate layers should return None
                self.assertIsNone(result, f"Layer {layer_idx} should return None")

        print("✓ Single layer processing works correctly")
```
**EN:** This range defines concrete test routine(s) `test_split_prefill_edge_cases`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `prepare_test_batch`, `forward_split_prefill`, `assertIsNotNone` and `assertIsNone`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 303-306: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Environment-aware configuration / 环境感知配置
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: `numpy`, `torch`
- **Project Modules / 项目模块**: `sglang.bench_one_batch`, `sglang.srt.configs.model_config`, `sglang.srt.managers.schedule_batch`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.model_executor.model_runner`, `sglang.srt.sampling.sampling_params`, `sglang.srt.server_args`, `sglang.srt.speculative.spec_info`, `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.test_utils`
