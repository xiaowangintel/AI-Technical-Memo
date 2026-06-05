# test_lora_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/lora/test_lora_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `lora ops` scenario in `test/manual/lora`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/lora` 中的 `lora ops` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Scenario logic / 场景逻辑
```python
import random
import unittest

import torch

from sglang.srt.lora.torch_ops.graph_lora_ops import (
    sgemm_lora_a_embedding_graph_fwd,
    sgemm_lora_a_graph_fwd,
    sgemm_lora_b_graph_fwd,
)
from sglang.srt.lora.torch_ops.lora_ops import (
    sgemm_lora_a_embedding_fwd,
    sgemm_lora_a_fwd,
    sgemm_lora_b_fwd,
)
from sglang.test.lora_utils import (
    reference_embedding_lora_a_shrink,
    reference_sgmv_expand,
    reference_sgmv_shrink,
)
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 24-24: Class definition for TestLoraOps / 类定义
```python
class TestLoraOps(CustomTestCase):
```
**EN:** This range declares `TestLoraOps`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 25-48: Test routines around test_sgemm_lora_a_embedding_fwd / 测试例程
```python
    def test_sgemm_lora_a_embedding_fwd(self):
        batch_size = 64
        input_dim = 1024
        num_loras = 3
        dtype = torch.float32
        vocab_size = 32000

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
            k=num_loras,
        )

        max_lora_rank = max(lora_ranks)

        possible_lora_scaling = [0.25, 0.5, 1.0, 2.0, 4.0]
        lora_scaling = random.sample(
            possible_lora_scaling,
            counts=[num_loras] * len(possible_lora_scaling),
            k=num_loras,
        )

        inputs = torch.randint(vocab_size, (batch_size,), dtype=torch.int32)
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_a_embedding_fwd`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `sample` and `randint`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 49-49: Scenario logic / 场景逻辑
```python
        lora_a_weights = torch.randn(num_loras, max_lora_rank, vocab_size, dtype=dtype)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-73: Scenario logic / 场景逻辑
```python
        lora_indices_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
        seq_len_tensor = torch.ones(batch_size, dtype=torch.int32, device="cpu")
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        lora_scaling_tensor = torch.tensor(
            lora_scaling, dtype=torch.float16, device="cpu"
        )

        expect_output = reference_embedding_lora_a_shrink(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
            vocab_size,
        )

        actual_output = sgemm_lora_a_embedding_fwd(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint`, `ones`, `tensor` and `reference_embedding_lora_a_shrink`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 74-74: Scenario logic / 场景逻辑
```python
            lora_ranks_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 75-79: Assertions and result checks / 断言与结果检查
```python
            lora_scaling_tensor,
            vocab_size,
        )

        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 81-98: Test routines around test_sgemm_lora_a_fwd / 测试例程
```python
    def test_sgemm_lora_a_fwd(self):
        batch_size = 2
        input_dim = 1024
        num_loras = 3
        dtype = torch.float32

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
            k=num_loras,
        )

        max_lora_rank = max(lora_ranks)

        possible_lora_scaling = [0.25, 0.5, 1.0, 2.0, 4.0]
        lora_scaling = random.sample(
            possible_lora_scaling,
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_a_fwd`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `sample`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 99-105: Scenario logic / 场景逻辑
```python
            counts=[num_loras] * len(possible_lora_scaling),
            k=num_loras,
        )

        inputs = torch.randn(batch_size, input_dim, dtype=dtype)
        lora_a_weights = torch.randn(num_loras, max_lora_rank, input_dim, dtype=dtype)
        lora_indices_tensor = torch.randint(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 106-123: Scenario logic / 场景逻辑
```python
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
        seq_len_tensor = torch.ones(batch_size, dtype=torch.int32, device="cpu")
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        lora_scaling_tensor = torch.tensor(
            lora_scaling, dtype=torch.float16, device="cpu"
        )

        expect_output = reference_sgmv_shrink(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
        )

        actual_output = sgemm_lora_a_fwd(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ones`, `tensor`, `reference_sgmv_shrink` and `sgemm_lora_a_fwd`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 124-130: Scenario logic / 场景逻辑
```python
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 131-132: Assertions and result checks / 断言与结果检查
```python

        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 134-148: Test routines around test_sgemm_lora_b_fwd / 测试例程
```python
    def test_sgemm_lora_b_fwd(self):
        batch_size = 2
        output_dim = 1024
        num_loras = 3
        dtype = torch.float32

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
            k=num_loras,
        )

        max_lora_rank = max(lora_ranks)
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_b_fwd`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `sample`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 149-158: Scenario logic / 场景逻辑
```python
        inputs = torch.randn(batch_size, max_lora_rank, dtype=dtype)
        lora_b_weights = torch.randn(num_loras, output_dim, max_lora_rank, dtype=dtype)
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        seq_len_tensor = torch.ones(batch_size, dtype=torch.int32, device="cpu")
        lora_indices_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
        slice_offsets = torch.tensor([0, output_dim], dtype=torch.int32, device="cpu")

        expect_output = reference_sgmv_expand(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `tensor`, `ones` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 159-173: Scenario logic / 场景逻辑
```python
            inputs,
            lora_b_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            slice_offsets,
        )

        actual_output = sgemm_lora_b_fwd(
            inputs,
            lora_b_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            slice_offsets,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sgemm_lora_b_fwd`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 174-176: Assertions and result checks / 断言与结果检查
```python
        )

        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 178-198: Test routines around test_sgemm_lora_a_embedding_fwd_expand / 测试例程
```python
    def test_sgemm_lora_a_embedding_fwd_expand(self):
        batch_size = 2
        input_dim = 1024
        num_loras = 3
        dtype = torch.float32
        vocab_size = 32000

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
            k=num_loras,
        )

        max_lora_rank = max(lora_ranks)

        possible_lora_scaling = [0.25, 0.5, 1.0, 2.0, 4.0]
        lora_scaling = random.sample(
            possible_lora_scaling,
            counts=[num_loras] * len(possible_lora_scaling),
            k=num_loras,
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_a_embedding_fwd_expand`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `sample`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 199-202: Scenario logic / 场景逻辑
```python
        )

        seq_len_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 203-223: Scenario logic / 场景逻辑
```python
        )

        seq_len = sum(seq_len_tensor)

        inputs = torch.randint(vocab_size, (seq_len,), dtype=torch.int32)
        lora_a_weights = torch.randn(num_loras, max_lora_rank, vocab_size, dtype=dtype)
        lora_indices_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        lora_scaling_tensor = torch.tensor(
            lora_scaling, dtype=torch.float16, device="cpu"
        )

        expect_output = reference_embedding_lora_a_shrink(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint`, `randn`, `tensor` and `reference_embedding_lora_a_shrink`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 224-227: Scenario logic / 场景逻辑
```python
            vocab_size,
        )

        actual_output = sgemm_lora_a_embedding_fwd(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sgemm_lora_a_embedding_fwd`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 228-237: Assertions and result checks / 断言与结果检查
```python
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
            vocab_size,
        )

        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 239-248: Test routines around test_sgemm_lora_a_fwd_expand / 测试例程
```python
    def test_sgemm_lora_a_fwd_expand(self):
        batch_size = 2
        input_dim = 1024
        num_loras = 3
        dtype = torch.float32

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_a_fwd_expand`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `sample`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 249-263: Scenario logic / 场景逻辑
```python
            k=num_loras,
        )

        max_lora_rank = max(lora_ranks)

        possible_lora_scaling = [0.25, 0.5, 1.0, 2.0, 4.0]
        lora_scaling = random.sample(
            possible_lora_scaling,
            counts=[num_loras] * len(possible_lora_scaling),
            k=num_loras,
        )

        seq_len_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sample` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 264-273: Scenario logic / 场景逻辑
```python

        seq_len = sum(seq_len_tensor)

        inputs = torch.randn(seq_len, input_dim, dtype=dtype)
        lora_a_weights = torch.randn(num_loras, max_lora_rank, input_dim, dtype=dtype)
        lora_indices_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        lora_scaling_tensor = torch.tensor(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `randint` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 274-288: Scenario logic / 场景逻辑
```python
            lora_scaling, dtype=torch.float16, device="cpu"
        )

        expect_output = reference_sgmv_shrink(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
        )

        actual_output = sgemm_lora_a_fwd(
            inputs,
            lora_a_weights,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `reference_sgmv_shrink` and `sgemm_lora_a_fwd`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 289-295: Assertions and result checks / 断言与结果检查
```python
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
        )

        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 297-298: Test routines around test_sgemm_lora_b_fwd_expand / 测试例程
```python
    def test_sgemm_lora_b_fwd_expand(self):
        batch_size = 2
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_b_fwd_expand`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 299-321: Scenario logic / 场景逻辑
```python
        output_dim = 1024
        num_loras = 3
        dtype = torch.float32

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
            k=num_loras,
        )

        max_lora_rank = max(lora_ranks)

        seq_len_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )

        seq_len = sum(seq_len_tensor)

        inputs = torch.randn(seq_len, max_lora_rank, dtype=dtype)
        lora_b_weights = torch.randn(num_loras, output_dim, max_lora_rank, dtype=dtype)
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        lora_indices_tensor = torch.randint(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sample`, `randint`, `randn` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 322-323: Scenario logic / 场景逻辑
```python
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 324-344: Assertions and result checks / 断言与结果检查
```python
        slice_offsets = torch.tensor([0, output_dim], dtype=torch.int32, device="cpu")

        expect_output = reference_sgmv_expand(
            inputs,
            lora_b_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            slice_offsets,
        )

        actual_output = sgemm_lora_b_fwd(
            inputs,
            lora_b_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            slice_offsets,
        )

        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tensor`, `reference_sgmv_expand`, `sgemm_lora_b_fwd` and `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 346-348: Test routines around test_sgemm_lora_a_embedding_graph_fwd / 测试例程
```python
    def test_sgemm_lora_a_embedding_graph_fwd(self):
        batch_size = 4
        input_dim = 1024
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_a_embedding_graph_fwd`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 349-370: Scenario logic / 场景逻辑
```python
        num_loras = 3
        dtype = torch.float16
        vocab_size = 32000

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
            k=num_loras,
        )

        max_lora_rank = max(lora_ranks)

        possible_lora_scaling = [0.25, 0.5, 1.0, 2.0, 4.0]
        lora_scaling = random.sample(
            possible_lora_scaling,
            counts=[num_loras] * len(possible_lora_scaling),
            k=num_loras,
        )

        inputs = torch.randint(vocab_size, (batch_size,), dtype=torch.int32)
        lora_a_weights = torch.zeros(num_loras, max_lora_rank, vocab_size, dtype=dtype)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sample`, `randint` and `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 371-373: Scenario logic / 场景逻辑
```python
        for idx, rank in enumerate(lora_ranks):
            lora_a_weights[idx, :rank] = torch.randn(rank, vocab_size, dtype=dtype)
        lora_indices_tensor = torch.randint(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 374-395: Scenario logic / 场景逻辑
```python
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
        seq_len_tensor = torch.ones(batch_size, dtype=torch.int32, device="cpu")
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        lora_scaling_tensor = torch.tensor(
            lora_scaling, dtype=torch.float16, device="cpu"
        )

        expect_output = reference_embedding_lora_a_shrink(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
            vocab_size,
        )

        actual_output = sgemm_lora_a_embedding_graph_fwd(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ones`, `tensor`, `reference_embedding_lora_a_shrink` and `sgemm_lora_a_embedding_graph_fwd`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 396-398: Scenario logic / 场景逻辑
```python
            seq_len_tensor,
            lora_scaling_tensor,
            vocab_size,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 399-403: Assertions and result checks / 断言与结果检查
```python
        )

        self.assertTrue(
            torch.allclose(actual_output, expect_output, rtol=1e-3, atol=1e-5)
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 405-423: Test routines around test_sgemm_lora_a_graph_fwd / 测试例程
```python
    def test_sgemm_lora_a_graph_fwd(self):
        batch_size = 4
        input_dim = 1024
        num_loras = 3
        dtype = torch.float16

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
            k=num_loras,
        )

        max_lora_rank = max(lora_ranks)

        possible_lora_scaling = [0.25, 0.5, 1.0, 2.0, 4.0]
        lora_scaling = random.sample(
            possible_lora_scaling,
            counts=[num_loras] * len(possible_lora_scaling),
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_a_graph_fwd`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `sample`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 424-429: Scenario logic / 场景逻辑
```python
            k=num_loras,
        )

        inputs = torch.randn(batch_size, input_dim, dtype=dtype)
        lora_a_weights = torch.zeros(num_loras, max_lora_rank, input_dim, dtype=dtype)
        for idx, rank in enumerate(lora_ranks):
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn` and `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 430-448: Scenario logic / 场景逻辑
```python
            lora_a_weights[idx, :rank] = torch.randn(rank, input_dim, dtype=dtype)
        lora_indices_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
        seq_len_tensor = torch.ones(batch_size, dtype=torch.int32, device="cpu")
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        lora_scaling_tensor = torch.tensor(
            lora_scaling, dtype=torch.float16, device="cpu"
        )

        expect_output = reference_sgmv_shrink(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            lora_scaling_tensor,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `randint`, `ones` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 449-454: Scenario logic / 场景逻辑
```python
        actual_output = sgemm_lora_a_graph_fwd(
            inputs,
            lora_a_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_scaling_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sgemm_lora_a_graph_fwd`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 455-459: Assertions and result checks / 断言与结果检查
```python
        )

        self.assertTrue(
            torch.allclose(actual_output, expect_output, rtol=1e-3, atol=1e-5)
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 461-473: Test routines around test_sgemm_lora_b_graph_fwd / 测试例程
```python
    def test_sgemm_lora_b_graph_fwd(self):
        batch_size = 4
        output_dim = 1024
        num_loras = 3
        dtype = torch.float16

        possible_lora_ranks = [8, 16, 32, 64, 128, 256]
        lora_ranks = random.sample(
            possible_lora_ranks,
            counts=[num_loras] * len(possible_lora_ranks),
            k=num_loras,
        )
```
**EN:** This range defines concrete test routine(s) `test_sgemm_lora_b_graph_fwd`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `sample`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 474-485: Scenario logic / 场景逻辑
```python
        max_lora_rank = max(lora_ranks)

        inputs = torch.randn(batch_size, max_lora_rank, dtype=dtype)
        lora_b_weights = torch.zeros(num_loras, output_dim, max_lora_rank, dtype=dtype)
        for idx, rank in enumerate(lora_ranks):
            lora_b_weights[idx, ..., :rank] = torch.randn(output_dim, rank, dtype=dtype)
        lora_ranks_tensor = torch.tensor(lora_ranks, dtype=torch.int32, device="cpu")
        seq_len_tensor = torch.ones(batch_size, dtype=torch.int32, device="cpu")
        lora_indices_tensor = torch.randint(
            num_loras, (batch_size,), dtype=torch.int32, device="cpu"
        )
        slice_offsets = torch.tensor([0, output_dim], dtype=torch.int32, device="cpu")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `zeros`, `tensor` and `ones`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 486-498: Scenario logic / 场景逻辑
```python

        expect_output = reference_sgmv_expand(
            inputs,
            lora_b_weights,
            lora_indices_tensor,
            seq_len_tensor,
            lora_ranks_tensor,
            slice_offsets,
        )

        actual_output = sgemm_lora_b_graph_fwd(
            inputs,
            lora_b_weights,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `reference_sgmv_expand` and `sgemm_lora_b_graph_fwd`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 499-506: Assertions and result checks / 断言与结果检查
```python
            lora_indices_tensor,
            seq_len_tensor,
            slice_offsets,
        )

        self.assertTrue(
            torch.allclose(actual_output, expect_output, rtol=1e-3, atol=1e-5)
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 507-510: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- LoRA adaptation / LoRA 适配

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `random`, `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.srt.lora.torch_ops.graph_lora_ops`, `sglang.srt.lora.torch_ops.lora_ops`, `sglang.test.lora_utils`, `sglang.test.test_utils`
