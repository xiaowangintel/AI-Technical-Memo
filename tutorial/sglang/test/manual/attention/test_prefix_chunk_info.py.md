# test_prefix_chunk_info.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/attention/test_prefix_chunk_info.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `prefix chunk info` scenario in `test/manual/attention`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/attention` 中的 `prefix chunk info` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-93: Constants and scenario settings / 常量与场景配置
```python
import unittest

import torch

from sglang.srt.mem_cache.memory_pool import MLATokenToKVPool
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.utils.common import get_device
from sglang.test.test_utils import CustomTestCase

TEST_CASES = [
    # Sequence with same prefix lens
    {
        "batch_size": 3,
        "prefix_lens": [64, 64, 64],
        "max_chunk_capacity": 48,
        "prefix_chunk_len": 16,
        "num_prefix_chunks": 4,
        "prefix_chunk_starts": torch.tensor(
            [
                [0, 0, 0],
                [16, 16, 16],
                [32, 32, 32],
                [48, 48, 48],
            ],
            dtype=torch.int32,
        ),
        "prefix_chunk_seq_lens": torch.tensor(
            [
                [16, 16, 16],
                [16, 16, 16],
                [16, 16, 16],
                [16, 16, 16],
            ],
            dtype=torch.int32,
        ),
    },
    # Sequence with different prefix lens
    {
        "batch_size": 4,
        "prefix_lens": [16, 32, 48, 64],
        "max_chunk_capacity": 64,
        "prefix_chunk_len": 16,
        "num_prefix_chunks": 4,
        "prefix_chunk_starts": torch.tensor(
            [
                [0, 0, 0, 0],
                [16, 16, 16, 16],
                [32, 32, 32, 32],
                [48, 48, 48, 48],
            ],
            dtype=torch.int32,
        ),
        "prefix_chunk_seq_lens": torch.tensor(
            [
                [16, 16, 16, 16],
                [0, 16, 16, 16],
                [0, 0, 16, 16],
                [0, 0, 0, 16],
            ],
            dtype=torch.int32,
        ),
    },
    # Sequence with irregular shapes
    {
        "batch_size": 2,
        "prefix_lens": [1, 64],
        "max_chunk_capacity": 31,
        "prefix_chunk_len": 15,
        "num_prefix_chunks": 5,
        "prefix_chunk_starts": torch.tensor(
            [
                [0, 0],
                [15, 15],
                [30, 30],
                [45, 45],
                [60, 60],
            ],
            dtype=torch.int32,
        ),
        "prefix_chunk_seq_lens": torch.tensor(
            [
                [1, 15],
                [0, 15],
                [0, 15],
                [0, 15],
                [0, 4],
            ],
            dtype=torch.int32,
        ),
    },
]
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `tensor`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 94-94: Class definition for MockForwardBatch / 类定义
```python
class MockForwardBatch(ForwardBatch):
```
**EN:** This range declares `MockForwardBatch`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 95-97: Helper routines around __init__ / 辅助例程
```python
    def __init__(self, max_chunk_capacity: int, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.max_chunk_capacity = max_chunk_capacity
```
**EN:** This range implements helper routine(s) `__init__` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 99-100: Helper routines around get_max_chunk_capacity / 辅助例程
```python
    def get_max_chunk_capacity(self):
        return self.max_chunk_capacity
```
**EN:** This range implements helper routine(s) `get_max_chunk_capacity` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 103-103: Class definition for MockReqToTokenPool / 类定义
```python
class MockReqToTokenPool:
```
**EN:** This range declares `MockReqToTokenPool`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 104-109: Helper routines around __init__ / 辅助例程
```python
    def __init__(self, batch_size, seq_len, device):
        self.req_to_token = (
            torch.arange(batch_size * seq_len, device=device)
            .reshape(batch_size, seq_len)
            .to(torch.int32)
        )
```
**EN:** This range implements helper routine(s) `__init__` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `arange`, `reshape` and `to`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 110-112: Scenario logic / 场景逻辑
```python


# Test correctness of triton kernel for computing kv indices
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 113-132: Helper routines around check_kv_indices / 辅助例程
```python
def check_kv_indices(forward_batch):
    for i in range(forward_batch.num_prefix_chunks):
        computed_kv_indices = forward_batch.prefix_chunk_kv_indices[i]
        req_to_token = forward_batch.req_to_token_pool.req_to_token[
            : forward_batch.batch_size, :
        ]
        ref_kv_indices = torch.empty(
            forward_batch.prefix_chunk_num_tokens[i],
            dtype=torch.int32,
            device=computed_kv_indices.device,
        )
        running_ptr = 0
        for j in range(forward_batch.batch_size):
            seq_start = forward_batch.prefix_chunk_starts[i, j].item()
            seq_len = forward_batch.prefix_chunk_seq_lens[i, j].item()
            ref_kv_indices[running_ptr : running_ptr + seq_len].copy_(
                req_to_token[j, seq_start : seq_start + seq_len]
            )
            running_ptr += seq_len
        assert torch.allclose(computed_kv_indices, ref_kv_indices)
```
**EN:** This range implements helper routine(s) `check_kv_indices` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `empty`, `item`, `copy_` and `allclose`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 133-138: Scenario logic / 场景逻辑
```python


@unittest.skipIf(
    not (torch.cuda.is_available() or torch.xpu.is_available()),
    "Test requires CUDA or XPU",
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipIf`, `not` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 139-139: Class definition for TestPrefixChunkInfo / 类定义
```python
class TestPrefixChunkInfo(CustomTestCase):
```
**EN:** This range declares `TestPrefixChunkInfo`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 140-163: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUp(self):
        # Common test parameters
        self.num_local_heads = 128
        self.kv_lora_rank = 512
        self.qk_rope_head_dim = 64
        self.device = get_device()
        self.dtype = torch.bfloat16
        self.extend_len = 64
        self.max_bs = 4
        self.max_seq_len = 128

        # req_to_token_pool
        self.req_to_token_pool = MockReqToTokenPool(
            self.max_bs,
            self.max_seq_len,
            self.device,
        )

        # token_to_kv_pool
        self.token_to_kv_pool = MLATokenToKVPool(
            size=self.max_bs * self.max_seq_len,
            page_size=1,  # only consider page=1 for unit test
            dtype=self.dtype,
            kv_lora_rank=self.kv_lora_rank,
```
**EN:** This range implements lifecycle helper(s) `setUp` to prepare or clean up shared resources across the test flow. Representative call sites include `get_device`, `MockReqToTokenPool` and `MLATokenToKVPool`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 164-164: Scenario logic / 场景逻辑
```python
            qk_rope_head_dim=self.qk_rope_head_dim,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 165-168: Scenario logic / 场景逻辑
```python
            layer_num=1,  # only consider layer=1 for unit test
            device=self.device,
            enable_memory_saver=False,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 170-188: Test routines around test_prefix_chunk_info / 测试例程
```python
    def test_prefix_chunk_info(self):
        """Test the standard extend operation."""

        for test_case in TEST_CASES:
            print(
                f"Test case with batch_size={test_case['batch_size']}, prefix_lens={test_case['prefix_lens']}, max_chunk_capacity={test_case['max_chunk_capacity']}"
            )
            batch_size = test_case["batch_size"]
            prefix_lens_cpu = test_case["prefix_lens"]
            assert len(prefix_lens_cpu) == batch_size
            prefix_lens = torch.tensor(prefix_lens_cpu, device=self.device)
            max_chunk_capacity = test_case["max_chunk_capacity"]
            seq_lens_cpu = [
                self.extend_len + prefix_lens_cpu[i] for i in range(batch_size)
            ]
            seq_lens = torch.tensor(seq_lens_cpu, device=self.device)

            # Create forward batch
            # input_ids and out_cache_loc are dummy tensors in this test
```
**EN:** This range defines concrete test routine(s) `test_prefix_chunk_info`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `tensor`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 189-194: Scenario logic / 场景逻辑
```python
            forward_batch = MockForwardBatch(
                max_chunk_capacity=max_chunk_capacity,
                batch_size=batch_size,
                input_ids=torch.randint(
                    0, 100, (batch_size, self.extend_len), device=self.device
                ),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `MockForwardBatch` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 195-213: Assertions and result checks / 断言与结果检查
```python
                out_cache_loc=torch.arange(
                    self.max_bs * self.max_seq_len - batch_size * self.extend_len,
                    self.max_bs * self.max_seq_len,
                    device=self.device,
                ),
                seq_lens_sum=sum(seq_lens_cpu),
                forward_mode=ForwardMode.EXTEND,
                req_pool_indices=torch.arange(batch_size, device=self.device),
                seq_lens=seq_lens,
                seq_lens_cpu=seq_lens_cpu,
                extend_prefix_lens=prefix_lens,
                extend_prefix_lens_cpu=prefix_lens_cpu,
            )
            forward_batch.req_to_token_pool = self.req_to_token_pool
            forward_batch.token_to_kv_pool = self.token_to_kv_pool

            forward_batch.prepare_chunked_prefix_cache_info(self.device)
            assert forward_batch.get_max_chunk_capacity() == max_chunk_capacity
            assert forward_batch.prefix_chunk_len == test_case["prefix_chunk_len"]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `arange`, `prepare_chunked_prefix_cache_info` and `get_max_chunk_capacity`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 214-219: Assertions and result checks / 断言与结果检查
```python
            assert forward_batch.num_prefix_chunks == test_case["num_prefix_chunks"]
            assert torch.allclose(
                forward_batch.prefix_chunk_starts,
                test_case["prefix_chunk_starts"].to(self.device),
            )
            assert torch.allclose(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `allclose` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 220-224: Scenario logic / 场景逻辑
```python
                forward_batch.prefix_chunk_seq_lens,
                test_case["prefix_chunk_seq_lens"].to(self.device),
            )

            check_kv_indices(forward_batch)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `to` and `check_kv_indices`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 225-228: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Token-level inspection / Token 级分析
- LoRA adaptation / LoRA 适配
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.srt.mem_cache.memory_pool`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.utils.common`, `sglang.test.test_utils`
