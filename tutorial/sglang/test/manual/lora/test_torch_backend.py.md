# test_torch_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/lora/test_torch_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `torch backend` scenario in `test/manual/lora`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/lora` 中的 `torch backend` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

import torch

from sglang.srt.lora.backend.torch_backend import TorchNativeLoRABackend
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.test.lora_utils import reference_sgmv_expand, reference_sgmv_shrink
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range imports `unittest`, `torch`, `sglang.srt.lora.backend.torch_backend` and `sglang.srt.model_executor.forward_batch_info`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 11-34: Class definition for TestTorchNativeLoRABackend / 类定义
```python
class TestTorchNativeLoRABackend(CustomTestCase):

    device = "cpu"

    # set duplicate weights to test merging during prepare_lora_batch
    weight_indices = [0, 0, 1]
    lora_ranks = [1, 1]
    scalings = [1.0, 0.5]
    seq_lens = [1, 1, 1]
    use_cuda_graph = False

    forward_batch = ForwardBatch(
        forward_mode=ForwardMode.EXTEND,
        batch_size=3,
        input_ids=torch.tensor([[1], [2], [3]], dtype=torch.int32),
        req_pool_indices=None,
        seq_lens=None,
        out_cache_loc=None,
        seq_lens_sum=3,
        extend_seq_lens=torch.tensor(seq_lens, dtype=torch.int32),
        extend_seq_lens_cpu=seq_lens,
    )

    @classmethod
```
**EN:** This range declares `TestTorchNativeLoRABackend`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `ForwardBatch` and `tensor`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 35-35: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。

### Lines 36-43: Scenario logic / 场景逻辑
```python
        cls.backend = TorchNativeLoRABackend(max_loras_per_batch=2, device=cls.device)
        cls.backend.prepare_lora_batch(
            forward_batch=cls.forward_batch,
            weight_indices=cls.weight_indices,
            lora_ranks=cls.lora_ranks,
            scalings=cls.scalings,
            use_cuda_graph=cls.use_cuda_graph,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `TorchNativeLoRABackend` and `prepare_lora_batch`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 45-60: Test routines around test_run_lora_a_sgemm / 测试例程
```python
    def test_run_lora_a_sgemm(self):
        batch_size = 3
        input_dim = 4
        output_dim = 6
        num_loras = 3
        dtype = torch.float32

        x = torch.randn(batch_size, input_dim, dtype=dtype)
        weights = torch.randn(num_loras, output_dim, input_dim, dtype=dtype)

        weight_indices_tensor = torch.tensor(
            self.weight_indices, dtype=torch.int32, device=self.device
        )

        seg_len_tensor = torch.tensor(
            self.seq_lens, dtype=torch.int32, device=self.device
```
**EN:** This range defines concrete test routine(s) `test_run_lora_a_sgemm`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `randn` and `tensor`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 61-69: Scenario logic / 场景逻辑
```python
        )

        lora_ranks_tensor = torch.tensor(
            self.lora_ranks, dtype=torch.int32, device=self.device
        )

        scalings_tensor = torch.tensor(
            self.scalings, dtype=torch.float, device=self.device
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 70-82: Assertions and result checks / 断言与结果检查
```python

        expect_output = reference_sgmv_shrink(
            x,
            weights,
            weight_indices_tensor,
            seg_len_tensor,
            lora_ranks_tensor,
            scalings_tensor,
        )

        actual_output = self.backend.run_lora_a_sgemm(x, weights)

        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `reference_sgmv_shrink`, `run_lora_a_sgemm`, `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 84-85: Test routines around test_run_lora_b_sgemm / 测试例程
```python
    def test_run_lora_b_sgemm(self):
        batch_size = 3
```
**EN:** This range defines concrete test routine(s) `test_run_lora_b_sgemm`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 86-108: Scenario logic / 场景逻辑
```python
        input_dim = 6
        output_dim = 4
        num_loras = 3
        dtype = torch.float32

        x = torch.randn(batch_size, input_dim, dtype=dtype)
        weights = torch.randn(num_loras, output_dim, input_dim, dtype=dtype)
        _, weight_out_dim, _ = weights.shape

        weight_indices_tensor = torch.tensor(
            self.weight_indices, dtype=torch.int32, device=self.device
        )

        seg_len_tensor = torch.tensor(
            self.seq_lens, dtype=torch.int32, device=self.device
        )

        lora_ranks_tensor = torch.tensor(
            self.lora_ranks, dtype=torch.int32, device=self.device
        )

        expect_output = reference_sgmv_expand(
            x,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `tensor` and `reference_sgmv_expand`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 109-110: Scenario logic / 场景逻辑
```python
            weights,
            weight_indices_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 111-120: Assertions and result checks / 断言与结果检查
```python
            seg_len_tensor,
            lora_ranks_tensor,
            slice_offsets=torch.tensor(
                [0, weight_out_dim], dtype=torch.int32, device="cpu"
            ),
        )

        actual_output = self.backend.run_lora_b_sgemm(x, weights)

        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tensor`, `run_lora_b_sgemm`, `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 122-135: Test routines around test_run_qkv_lora / 测试例程
```python
    def test_run_qkv_lora(self):
        batch_size = 3
        num_loras = 3
        input_dim = 6
        output_offset = [0, 3, 6, 9]
        output_dim = output_offset[-1]
        num_slices = len(output_offset) - 1  # 3 slices for Q, K, V
        max_lora_rank = max(self.lora_ranks)
        dtype = torch.float32

        x = torch.randn(batch_size, input_dim, dtype=dtype)
        output_offset_cpu = torch.tensor(output_offset, dtype=torch.int32)
        qkv_lora_a = torch.randn(
            num_loras, max_lora_rank * num_slices, input_dim, dtype=dtype
```
**EN:** This range defines concrete test routine(s) `test_run_qkv_lora`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `randn` and `tensor`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 136-146: Scenario logic / 场景逻辑
```python
        )
        qkv_lora_b = torch.randn(
            num_loras, output_dim, max_lora_rank * num_slices, dtype=dtype
        )

        weight_indices_tensor = torch.tensor(
            self.weight_indices, dtype=torch.int32, device=self.device
        )

        seg_len_tensor = torch.tensor(
            self.seq_lens, dtype=torch.int32, device=self.device
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 147-160: Scenario logic / 场景逻辑
```python
        )

        lora_ranks_tensor = torch.tensor(
            self.lora_ranks, dtype=torch.int32, device=self.device
        )

        scalings_tensor = torch.tensor(
            self.scalings, dtype=torch.float, device=self.device
        )

        expect_lora_a_output = reference_sgmv_shrink(
            x,
            qkv_lora_a,
            weight_indices_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor` and `reference_sgmv_shrink`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 161-171: Scenario logic / 场景逻辑
```python
            seg_len_tensor,
            lora_ranks_tensor,
            scalings_tensor,
            num_slices,
        )

        expect_output = reference_sgmv_expand(
            expect_lora_a_output,
            qkv_lora_b,
            weight_indices_tensor,
            seg_len_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `reference_sgmv_expand`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 172-179: Assertions and result checks / 断言与结果检查
```python
            lora_ranks_tensor,
            output_offset_cpu,
        )

        actual_output = self.backend.run_qkv_lora(
            x, qkv_lora_a, qkv_lora_b, None, output_offset_cpu, 0
        )
        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `run_qkv_lora`, `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 181-185: Test routines around test_run_gate_up_lora / 测试例程
```python
    def test_run_gate_up_lora(self):
        batch_size = 3
        input_dim = 6
        output_dim = 4
        num_loras = 3
```
**EN:** This range defines concrete test routine(s) `test_run_gate_up_lora`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 186-205: Scenario logic / 场景逻辑
```python
        dtype = torch.float32

        max_lora_rank = max(self.lora_ranks)

        num_slices = 2

        x = torch.randn(batch_size, input_dim, dtype=dtype)
        gate_up_lora_a = torch.randn(
            num_loras, max_lora_rank * num_slices, input_dim, dtype=dtype
        )
        gate_up_lora_b = torch.randn(
            num_loras, output_dim, max_lora_rank * num_slices, dtype=dtype
        )

        _, weight_out_dim, _ = gate_up_lora_b.shape
        slice_size = weight_out_dim // num_slices
        output_offset = torch.tensor(
            [0, slice_size, weight_out_dim], dtype=torch.int32, device="cpu"
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 206-210: Scenario logic / 场景逻辑
```python
        weight_indices_tensor = torch.tensor(
            self.weight_indices, dtype=torch.int32, device=self.device
        )

        seg_len_tensor = torch.tensor(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 211-230: Scenario logic / 场景逻辑
```python
            self.seq_lens, dtype=torch.int32, device=self.device
        )

        lora_ranks_tensor = torch.tensor(
            self.lora_ranks, dtype=torch.int32, device=self.device
        )

        scalings_tensor = torch.tensor(
            self.scalings, dtype=torch.float, device=self.device
        )

        expect_lora_a_output = reference_sgmv_shrink(
            x,
            gate_up_lora_a,
            weight_indices_tensor,
            seg_len_tensor,
            lora_ranks_tensor,
            scalings_tensor,
            num_slices,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor` and `reference_sgmv_shrink`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 231-235: Scenario logic / 场景逻辑
```python

        expect_output = reference_sgmv_expand(
            expect_lora_a_output,
            gate_up_lora_b,
            weight_indices_tensor,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `reference_sgmv_expand`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 236-242: Assertions and result checks / 断言与结果检查
```python
            seg_len_tensor,
            lora_ranks_tensor,
            slice_offsets=output_offset,
        )

        actual_output = self.backend.run_gate_up_lora(x, gate_up_lora_a, gate_up_lora_b)
        self.assertTrue(torch.allclose(actual_output, expect_output))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `run_gate_up_lora`, `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 243-246: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- LoRA adaptation / LoRA 适配
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.srt.lora.backend.torch_backend`, `sglang.srt.model_executor.forward_batch_info`, `sglang.test.lora_utils`, `sglang.test.test_utils`
