# test_trtllm_fp8_kv_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_trtllm_fp8_kv_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `trtllm fp8 kv kernel` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual` 中的 `trtllm fp8 kv kernel` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and shared helpers / 导入与共享辅助项
```python
"""
Unit tests for TRTLLM FP8 KV cache fusion kernel.
"""

import unittest

import torch

from sglang.srt.layers.attention.triton_ops.trtllm_fp8_kv_kernel import (
    fused_fp8_set_kv_buffer,
)
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range imports `unittest`, `torch`, `sglang.srt.layers.attention.triton_ops.trtllm_fp8_kv_kernel` and `sglang.test.test_utils`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 15-18: Class definition for TestTRTLLMFP8KVKernel / 类定义
```python
class TestTRTLLMFP8KVKernel(CustomTestCase):
    """Test fused FP8 KV cache write kernel correctness."""

    @classmethod
```
**EN:** This range declares `TestTRTLLMFP8KVKernel`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 19-24: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA not available")

        if torch.cuda.get_device_capability()[0] < 9:
            raise unittest.SkipTest("FP8 requires compute capability >= 9.0")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `get_device_capability`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 26-39: Helper routines around _test_kernel_correctness / 辅助例程
```python
    def _test_kernel_correctness(
        self,
        num_tokens,
        num_kv_heads,
        head_dim,
        page_size,
        use_scale,
        input_ndim,
        cache_ndim,
    ):
        """Compare Triton kernel output against naive implementation."""
        device = torch.device("cuda")
        dtype = torch.bfloat16
```
**EN:** This range implements helper routine(s) `_test_kernel_correctness` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `device`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 40-50: Scenario logic / 场景逻辑
```python
        # Create input tensors
        if input_ndim == 3:
            k = torch.randn(
                num_tokens, num_kv_heads, head_dim, device=device, dtype=dtype
            )
            v = torch.randn(
                num_tokens, num_kv_heads, head_dim, device=device, dtype=dtype
            )
        else:
            k = torch.randn(
                num_tokens, num_kv_heads * head_dim, device=device, dtype=dtype
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 51-64: Scenario logic / 场景逻辑
```python
            )
            v = torch.randn(
                num_tokens, num_kv_heads * head_dim, device=device, dtype=dtype
            )

        # Create cache tensors (use FP8 to match real runtime behavior)
        num_pages = 128
        total_slots = num_pages * page_size
        cache_dtype = torch.float8_e4m3fn
        if cache_ndim == 3:
            k_cache_triton = torch.zeros(
                total_slots, num_kv_heads, head_dim, device=device, dtype=cache_dtype
            )
            v_cache_triton = torch.zeros(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `tensors` and `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-75: Scenario logic / 场景逻辑
```python
                total_slots, num_kv_heads, head_dim, device=device, dtype=cache_dtype
            )
            k_cache_naive = torch.zeros(
                total_slots, num_kv_heads, head_dim, device=device, dtype=cache_dtype
            )
            v_cache_naive = torch.zeros(
                total_slots, num_kv_heads, head_dim, device=device, dtype=cache_dtype
            )
        else:
            k_cache_triton = torch.zeros(
                num_pages,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 76-89: Scenario logic / 场景逻辑
```python
                page_size,
                num_kv_heads,
                head_dim,
                device=device,
                dtype=cache_dtype,
            )
            v_cache_triton = torch.zeros(
                num_pages,
                page_size,
                num_kv_heads,
                head_dim,
                device=device,
                dtype=cache_dtype,
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-100: Scenario logic / 场景逻辑
```python
            k_cache_naive = torch.zeros(
                num_pages,
                page_size,
                num_kv_heads,
                head_dim,
                device=device,
                dtype=cache_dtype,
            )
            v_cache_naive = torch.zeros(
                num_pages,
                page_size,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 101-114: Scenario logic / 场景逻辑
```python
                num_kv_heads,
                head_dim,
                device=device,
                dtype=cache_dtype,
            )

        # Create cache locations (ensure unique indices to avoid race conditions)
        cache_loc = torch.randperm(total_slots, device=device, dtype=torch.int32)[
            :num_tokens
        ]

        # Optional scales
        k_scale = 0.5 if use_scale else None
        v_scale = 0.75 if use_scale else None
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `locations` and `randperm`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 115-125: Scenario logic / 场景逻辑
```python

        # Run Triton kernel
        fused_fp8_set_kv_buffer(
            k.clone(),
            v.clone(),
            k_cache_triton,
            v_cache_triton,
            cache_loc,
            k_scale,
            v_scale,
            page_size,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `fused_fp8_set_kv_buffer` and `clone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 126-139: Scenario logic / 场景逻辑
```python
            use_triton=True,
        )

        # Run naive fallback
        fused_fp8_set_kv_buffer(
            k.clone(),
            v.clone(),
            k_cache_naive,
            v_cache_naive,
            cache_loc,
            k_scale,
            v_scale,
            page_size,
            use_triton=False,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `fused_fp8_set_kv_buffer` and `clone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 140-150: Assertions and result checks / 断言与结果检查
```python
        )

        # Compare results (bit-exact match expected)
        self.assertTrue(
            torch.equal(k_cache_triton, k_cache_naive),
            "K cache mismatch between Triton and naive",
        )
        self.assertTrue(
            torch.equal(v_cache_triton, v_cache_naive),
            "V cache mismatch between Triton and naive",
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `results`, `assertTrue` and `equal`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 152-162: Test routines around test_basic_3d_input_3d_cache / 测试例程
```python
    def test_basic_3d_input_3d_cache(self):
        """Test basic case: 3D input, 3D cache, no scale."""
        self._test_kernel_correctness(
            num_tokens=16,
            num_kv_heads=8,
            head_dim=128,
            page_size=16,
            use_scale=False,
            input_ndim=3,
            cache_ndim=3,
        )
```
**EN:** This range defines concrete test routine(s) `test_basic_3d_input_3d_cache`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_test_kernel_correctness`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 164-164: Test routines around test_basic_3d_input_4d_cache / 测试例程
```python
    def test_basic_3d_input_4d_cache(self):
```
**EN:** This range defines concrete test routine(s) `test_basic_3d_input_4d_cache`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 165-174: Scenario logic / 场景逻辑
```python
        """Test basic case: 3D input, 4D cache, no scale."""
        self._test_kernel_correctness(
            num_tokens=16,
            num_kv_heads=8,
            head_dim=128,
            page_size=16,
            use_scale=False,
            input_ndim=3,
            cache_ndim=4,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_test_kernel_correctness`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 176-186: Test routines around test_with_scale_3d_cache / 测试例程
```python
    def test_with_scale_3d_cache(self):
        """Test with scale: 3D input, 3D cache."""
        self._test_kernel_correctness(
            num_tokens=16,
            num_kv_heads=8,
            head_dim=128,
            page_size=16,
            use_scale=True,
            input_ndim=3,
            cache_ndim=3,
        )
```
**EN:** This range defines concrete test routine(s) `test_with_scale_3d_cache`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_test_kernel_correctness`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 188-189: Test routines around test_with_scale_4d_cache / 测试例程
```python
    def test_with_scale_4d_cache(self):
        """Test with scale: 3D input, 4D cache."""
```
**EN:** This range defines concrete test routine(s) `test_with_scale_4d_cache`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 190-198: Scenario logic / 场景逻辑
```python
        self._test_kernel_correctness(
            num_tokens=16,
            num_kv_heads=8,
            head_dim=128,
            page_size=16,
            use_scale=True,
            input_ndim=3,
            cache_ndim=4,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_test_kernel_correctness`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 200-210: Test routines around test_2d_input_3d_cache / 测试例程
```python
    def test_2d_input_3d_cache(self):
        """Test 2D input (flattened): 2D input, 3D cache."""
        self._test_kernel_correctness(
            num_tokens=16,
            num_kv_heads=8,
            head_dim=128,
            page_size=16,
            use_scale=False,
            input_ndim=2,
            cache_ndim=3,
        )
```
**EN:** This range defines concrete test routine(s) `test_2d_input_3d_cache`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `input` and `_test_kernel_correctness`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 212-214: Test routines around test_2d_input_4d_cache / 测试例程
```python
    def test_2d_input_4d_cache(self):
        """Test 2D input (flattened): 2D input, 4D cache."""
        self._test_kernel_correctness(
```
**EN:** This range defines concrete test routine(s) `test_2d_input_4d_cache`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `input` and `_test_kernel_correctness`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 215-222: Scenario logic / 场景逻辑
```python
            num_tokens=16,
            num_kv_heads=8,
            head_dim=128,
            page_size=16,
            use_scale=False,
            input_ndim=2,
            cache_ndim=4,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 224-234: Test routines around test_single_token / 测试例程
```python
    def test_single_token(self):
        """Test edge case: single token."""
        self._test_kernel_correctness(
            num_tokens=1,
            num_kv_heads=8,
            head_dim=128,
            page_size=16,
            use_scale=True,
            input_ndim=3,
            cache_ndim=3,
        )
```
**EN:** This range defines concrete test routine(s) `test_single_token`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_test_kernel_correctness`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 236-239: Test routines around test_large_batch / 测试例程
```python
    def test_large_batch(self):
        """Test larger batch size."""
        self._test_kernel_correctness(
            num_tokens=128,
```
**EN:** This range defines concrete test routine(s) `test_large_batch`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_test_kernel_correctness`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 240-246: Scenario logic / 场景逻辑
```python
            num_kv_heads=16,
            head_dim=64,
            page_size=16,
            use_scale=True,
            input_ndim=3,
            cache_ndim=4,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 248-259: Test routines around test_different_head_dims / 测试例程
```python
    def test_different_head_dims(self):
        """Test different head dimensions."""
        for head_dim in [64, 128]:
            self._test_kernel_correctness(
                num_tokens=16,
                num_kv_heads=8,
                head_dim=head_dim,
                page_size=16,
                use_scale=False,
                input_ndim=3,
                cache_ndim=3,
            )
```
**EN:** This range defines concrete test routine(s) `test_different_head_dims`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_test_kernel_correctness`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 261-264: Test routines around test_empty_input / 测试例程
```python
    def test_empty_input(self):
        """Test edge case: empty input (0 tokens)."""
        device = torch.device("cuda")
        dtype = torch.bfloat16
```
**EN:** This range defines concrete test routine(s) `test_empty_input`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `input` and `device`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 265-285: Scenario logic / 场景逻辑
```python
        num_kv_heads = 8
        head_dim = 128
        page_size = 16
        num_tokens = 0

        # Empty inputs
        k = torch.randn(num_tokens, num_kv_heads, head_dim, device=device, dtype=dtype)
        v = torch.randn(num_tokens, num_kv_heads, head_dim, device=device, dtype=dtype)

        # Cache (use FP8 to match real runtime behavior)
        total_slots = 128
        k_cache = torch.zeros(
            total_slots,
            num_kv_heads,
            head_dim,
            device=device,
            dtype=torch.float8_e4m3fn,
        )
        v_cache = torch.zeros(
            total_slots,
            num_kv_heads,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `Cache` and `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 286-289: Scenario logic / 场景逻辑
```python
            head_dim,
            device=device,
            dtype=torch.float8_e4m3fn,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 290-304: Scenario logic / 场景逻辑
```python

        # Empty cache locations
        cache_loc = torch.empty(num_tokens, device=device, dtype=torch.int32)

        # Should not crash
        fused_fp8_set_kv_buffer(
            k,
            v,
            k_cache,
            v_cache,
            cache_loc,
            k_scale=None,
            v_scale=None,
            page_size=page_size,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `empty` and `fused_fp8_set_kv_buffer`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 306-314: Test routines around test_fp8_kv_kernel_accepts_tensor_scales / 测试例程
```python
    def test_fp8_kv_kernel_accepts_tensor_scales(self):
        """
        Regression test for B200 Triton compilation issue.

        This test ensures that fused_fp8_set_kv_buffer correctly handles
        k_scale/v_scale when they are 0-dimensional tensors (torch.nn.Parameter).

        Previously, Triton would treat 0-D tensor arguments as pointers,
        causing a type error when performing "1.0 / k_scale" inside the kernel.
```
**EN:** This range defines concrete test routine(s) `test_fp8_kv_kernel_accepts_tensor_scales`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `tensors`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 315-330: Scenario logic / 场景逻辑
```python
        The fix converts tensor scales to Python floats in the wrapper.
        """
        device = torch.device("cuda")

        num_tokens = 4
        num_kv_heads = 2
        head_dim = 64
        page_size = 16
        total_slots = page_size

        k = torch.randn(
            num_tokens, num_kv_heads, head_dim, device=device, dtype=torch.bfloat16
        )
        v = torch.randn_like(k)

        k_cache = torch.empty(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `device`, `randn`, `randn_like` and `empty`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 331-339: Scenario logic / 场景逻辑
```python
            total_slots,
            num_kv_heads,
            head_dim,
            device=device,
            dtype=torch.float8_e4m3fn,
        )
        v_cache = torch.empty_like(k_cache)

        cache_loc = torch.arange(num_tokens, device=device, dtype=torch.int32)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `empty_like` and `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 340-355: Scenario logic / 场景逻辑
```python

        # Use 0D tensor form of scale to reproduce the original bug scenario
        k_scale = torch.tensor(1.0, device=device, dtype=torch.float32)
        v_scale = torch.tensor(1.0, device=device, dtype=torch.float32)

        # Old code would trigger Triton's IncompatibleTypeError here
        # New code should handle this gracefully by converting to float
        fused_fp8_set_kv_buffer(
            k,
            v,
            k_cache,
            v_cache,
            cache_loc,
            k_scale=k_scale,
            v_scale=v_scale,
            page_size=page_size,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor` and `fused_fp8_set_kv_buffer`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 356-357: Scenario logic / 场景逻辑
```python
            use_triton=True,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 358-360: Scenario logic / 场景逻辑
```python

        # If we get here without exception, the regression is fixed
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 361-364: Test routines around test_fp8_kv_kernel_cuda_graph_compatible / 测试例程
```python
    def test_fp8_kv_kernel_cuda_graph_compatible(self):
        """
        Regression test for CUDA graph capture compatibility.
```
**EN:** This range defines concrete test routine(s) `test_fp8_kv_kernel_cuda_graph_compatible`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 365-385: Scenario logic / 场景逻辑
```python
        This test ensures that fused_fp8_set_kv_buffer works correctly within
        CUDA graph capture, which is used in production for performance.

        Previously, float(k_scale) caused GPU→CPU synchronization, triggering
        cudaErrorStreamCaptureUnsupported during graph capture. The fix computes
        inverse scales purely on GPU using tensor operations.
        """
        device = torch.device("cuda")

        num_tokens = 4
        num_kv_heads = 2
        head_dim = 64
        page_size = 16
        total_slots = page_size

        k = torch.randn(
            num_tokens, num_kv_heads, head_dim, device=device, dtype=torch.bfloat16
        )
        v = torch.randn_like(k)

        k_cache = torch.empty(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `device`, `randn`, `randn_like` and `empty`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 386-389: Scenario logic / 场景逻辑
```python
            total_slots,
            num_kv_heads,
            head_dim,
            device=device,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 390-410: Scenario logic / 场景逻辑
```python
            dtype=torch.float8_e4m3fn,
        )
        v_cache = torch.empty_like(k_cache)

        cache_loc = torch.arange(num_tokens, device=device, dtype=torch.int32)

        # Use 0D tensor scales (like nn.Parameter) to reproduce production scenario
        k_scale = torch.tensor(1.0, device=device, dtype=torch.float32)
        v_scale = torch.tensor(1.0, device=device, dtype=torch.float32)

        # Test that kernel works under CUDA graph capture
        graph = torch.cuda.CUDAGraph()
        with torch.cuda.graph(graph):
            # Old code would fail here with cudaErrorStreamCaptureUnsupported
            # New code should succeed because all operations stay on GPU
            fused_fp8_set_kv_buffer(
                k,
                v,
                k_cache,
                v_cache,
                cache_loc,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `empty_like`, `arange`, `scales` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 411-414: Scenario logic / 场景逻辑
```python
                k_scale=k_scale,
                v_scale=v_scale,
                page_size=page_size,
                use_triton=True,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 415-418: Scenario logic / 场景逻辑
```python
            )

        # Replay the graph to verify it works
        graph.replay()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `replay`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 419-421: Scenario logic / 场景逻辑
```python

        # If we get here without exception, CUDA graph compatibility is confirmed
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 422-439: Test routines around test_fp8_kv_kernel_cuda_graph_compatible_no_scale / 测试例程
```python
    def test_fp8_kv_kernel_cuda_graph_compatible_no_scale(self):
        """
        Regression test for CUDA graph capture compatibility without scales.

        This test ensures that fused_fp8_set_kv_buffer works correctly within
        CUDA graph capture when k_scale/v_scale are None (use_provided_scale=False).

        Previously, the code created new GPU tensors (torch.tensor(1.0, device=...))
        during graph capture, triggering cudaErrorStreamCaptureUnsupported.
        The fix passes dummy pointers when use_provided_scale=False, as the kernel
        uses constant 1.0 and Triton optimizes away the pointer loads.
        """
        device = torch.device("cuda")

        num_tokens = 4
        num_kv_heads = 2
        head_dim = 64
        page_size = 16
```
**EN:** This range defines concrete test routine(s) `test_fp8_kv_kernel_cuda_graph_compatible_no_scale`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `None`, `tensors`, `tensor` and `device`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 440-446: Scenario logic / 场景逻辑
```python
        total_slots = page_size

        k = torch.randn(
            num_tokens, num_kv_heads, head_dim, device=device, dtype=torch.bfloat16
        )
        v = torch.randn_like(k)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn` and `randn_like`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 447-464: Scenario logic / 场景逻辑
```python
        k_cache = torch.empty(
            total_slots,
            num_kv_heads,
            head_dim,
            device=device,
            dtype=torch.float8_e4m3fn,
        )
        v_cache = torch.empty_like(k_cache)

        cache_loc = torch.arange(num_tokens, device=device, dtype=torch.int32)

        # Test that kernel works under CUDA graph capture WITHOUT scales
        graph = torch.cuda.CUDAGraph()
        with torch.cuda.graph(graph):
            # No k_scale/v_scale provided - use_provided_scale=False branch
            # Old code would fail here with cudaErrorStreamCaptureUnsupported
            # New code should succeed by using dummy pointers
            fused_fp8_set_kv_buffer(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `empty`, `empty_like`, `arange` and `CUDAGraph`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 465-471: Scenario logic / 场景逻辑
```python
                k,
                v,
                k_cache,
                v_cache,
                cache_loc,
                page_size=page_size,
                use_triton=True,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 472-475: Scenario logic / 场景逻辑
```python
            )

        # Replay the graph to verify it works
        graph.replay()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `replay`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 476-481: Script entry point / 脚本入口
```python

        # If we get here without exception, no-scale CUDA graph compatibility is confirmed


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Streaming responses / 流式响应
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.attention.triton_ops.trtllm_fp8_kv_kernel`, `sglang.test.test_utils`
