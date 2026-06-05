# test_index_buf_accessor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/layers/attention/nsa/test_index_buf_accessor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `index buf accessor` scenario in `test/manual/layers/attention/nsa`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/layers/attention/nsa` 中的 `index buf accessor` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Scenario logic / 场景逻辑
```python
"""
Correctness tests for NSA Indexer K/S Buffer Access with Fused Triton Kernels.

This test verifies that the optimized Triton implementations (GetK, GetS, GetKAndS)
produce identical results to the torch_fast baseline implementations.

Test coverage:
- GetK.triton() vs GetK.torch_fast()
- GetS.triton() vs GetS.torch_fast()
- GetKAndS.triton() vs separate GetK.torch_fast() + GetS.torch_fast()
"""

import pytest
import torch

from sglang.srt.layers.attention.nsa.index_buf_accessor import GetK, GetKAndS, GetS
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `implementations`, `triton` and `torch_fast`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 19-21: Class definition for MockNSATokenToKVPool / 类定义
```python
class MockNSATokenToKVPool:
    """Mock pool object that mimics NSATokenToKVPool for testing."""
```
**EN:** This range declares `MockNSATokenToKVPool`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 22-32: Helper routines around __init__ / 辅助例程
```python
    def __init__(
        self,
        page_size: int = 64,
        index_head_dim: int = 128,
        quant_block_size: int = 128,
        device: str = "cuda",
    ):
        self.page_size = page_size
        self.index_head_dim = index_head_dim
        self.quant_block_size = quant_block_size
        self.device = device
```
**EN:** This range implements helper routine(s) `__init__` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 35-59: Helper routines around create_test_buffer / 辅助例程
```python
def create_test_buffer(
    num_pages: int,
    page_size: int = 64,
    index_head_dim: int = 128,
    device: str = "cuda",
) -> torch.Tensor:
    """
    Create a test buffer mimicking the K/S buffer structure.

    Buffer layout per page:
    - First page_size * index_head_dim bytes: K data (fp8, stored as uint8)
    - Next page_size * 4 bytes: S data (fp32 scales, stored as uint8)

    Args:
        num_pages: Number of pages to allocate
        page_size: Tokens per page (typically 64)
        index_head_dim: Dimension of K vectors (typically 128)
        device: Device to allocate on

    Returns:
        Buffer of shape (num_pages, page_size * index_head_dim + page_size * 4)
    """
    buf_numel_per_page = page_size * index_head_dim + page_size * 4
    buf = torch.randint(
        0, 256, (num_pages, buf_numel_per_page), dtype=torch.uint8, device=device
```
**EN:** This range implements helper routine(s) `create_test_buffer` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `data`, `page`, `vectors` and `shape`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 60-61: Scenario logic / 场景逻辑
```python
    )
    return buf
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 62-64: Scenario logic / 场景逻辑
```python


@pytest.mark.skipif(not torch.cuda.is_available(), reason="CUDA not available")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipif` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-71: Class definition for TestGetK / 类定义
```python
class TestGetK:
    """Test cases for GetK.triton() correctness."""

    @pytest.mark.parametrize("num_pages", [1, 2, 4, 8, 16])
    @pytest.mark.parametrize("seq_len", [64, 128, 256, 512, 1024])
    @pytest.mark.parametrize("page_size", [64])
    @pytest.mark.parametrize("index_head_dim", [128])
```
**EN:** This range declares `TestGetK`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `triton` and `parametrize`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 72-89: Test routines around test_getk_correctness / 测试例程
```python
    def test_getk_correctness(self, num_pages, seq_len, page_size, index_head_dim):
        """Test GetK.triton() produces same output as GetK.torch_fast()."""
        device = torch.device("cuda")

        # Ensure seq_len doesn't exceed available pages
        max_seq_len = num_pages * page_size
        seq_len = min(seq_len, max_seq_len)

        # Create mock pool
        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )

        # Create test buffer
        buf = create_test_buffer(
            num_pages=num_pages,
            page_size=page_size,
            index_head_dim=index_head_dim,
```
**EN:** This range defines concrete test routine(s) `test_getk_correctness`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `triton`, `torch_fast`, `device` and `MockNSATokenToKVPool`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-96: Scenario logic / 场景逻辑
```python
            device=device,
        )

        # Create page indices
        num_pages_needed = (seq_len + page_size - 1) // page_size
        page_indices = torch.randint(
            0, num_pages, (num_pages_needed,), dtype=torch.int32, device=device
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 97-112: Assertions and result checks / 断言与结果检查
```python
        )

        # Run both implementations
        output_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        output_triton = GetK.triton(pool, buf, seq_len, page_indices)

        # Verify shapes
        assert output_torch.shape == (seq_len, index_head_dim)
        assert output_triton.shape == (seq_len, index_head_dim)
        assert output_torch.dtype == torch.uint8
        assert output_triton.dtype == torch.uint8

        # Compare results (should be exact match)
        torch.testing.assert_close(
            output_triton, output_torch, rtol=0, atol=0, msg="GetK outputs differ"
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `torch_fast`, `triton`, `results` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 114-114: Test routines around test_getk_sequential_pages / 测试例程
```python
    def test_getk_sequential_pages(self):
```
**EN:** This range defines concrete test routine(s) `test_getk_sequential_pages`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 115-133: Assertions and result checks / 断言与结果检查
```python
        """Test GetK with sequential page indices."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
        num_pages = 10
        seq_len = 320  # 5 pages

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)

        # Sequential page indices [0, 1, 2, 3, 4]
        page_indices = torch.arange(5, dtype=torch.int32, device=device)

        output_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        output_triton = GetK.triton(pool, buf, seq_len, page_indices)

        torch.testing.assert_close(output_triton, output_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `device`, `MockNSATokenToKVPool`, `create_test_buffer` and `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 135-139: Test routines around test_getk_repeated_pages / 测试例程
```python
    def test_getk_repeated_pages(self):
        """Test GetK with repeated page indices."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
```
**EN:** This range defines concrete test routine(s) `test_getk_repeated_pages`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 140-154: Assertions and result checks / 断言与结果检查
```python
        num_pages = 5
        seq_len = 192  # 3 pages

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)

        # Repeated page indices [2, 2, 2]
        page_indices = torch.full((3,), 2, dtype=torch.int32, device=device)

        output_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        output_triton = GetK.triton(pool, buf, seq_len, page_indices)

        torch.testing.assert_close(output_triton, output_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `MockNSATokenToKVPool`, `create_test_buffer`, `full` and `torch_fast`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 155-157: Scenario logic / 场景逻辑
```python


@pytest.mark.skipif(not torch.cuda.is_available(), reason="CUDA not available")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipif` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 158-164: Class definition for TestGetS / 类定义
```python
class TestGetS:
    """Test cases for GetS.triton() correctness."""

    @pytest.mark.parametrize("num_pages", [1, 2, 4, 8, 16])
    @pytest.mark.parametrize("seq_len", [64, 128, 256, 512, 1024])
    @pytest.mark.parametrize("page_size", [64])
    @pytest.mark.parametrize("index_head_dim", [128])
```
**EN:** This range declares `TestGetS`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `triton` and `parametrize`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 165-182: Test routines around test_gets_correctness / 测试例程
```python
    def test_gets_correctness(self, num_pages, seq_len, page_size, index_head_dim):
        """Test GetS.triton() produces same output as GetS.torch_fast()."""
        device = torch.device("cuda")

        # Ensure seq_len doesn't exceed available pages
        max_seq_len = num_pages * page_size
        seq_len = min(seq_len, max_seq_len)

        # Create mock pool
        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )

        # Create test buffer
        buf = create_test_buffer(
            num_pages=num_pages,
            page_size=page_size,
            index_head_dim=index_head_dim,
```
**EN:** This range defines concrete test routine(s) `test_gets_correctness`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `triton`, `torch_fast`, `device` and `MockNSATokenToKVPool`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 183-189: Scenario logic / 场景逻辑
```python
            device=device,
        )

        # Create page indices
        num_pages_needed = (seq_len + page_size - 1) // page_size
        page_indices = torch.randint(
            0, num_pages, (num_pages_needed,), dtype=torch.int32, device=device
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 190-205: Assertions and result checks / 断言与结果检查
```python
        )

        # Run both implementations
        output_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)
        output_triton = GetS.triton(pool, buf, seq_len, page_indices)

        # Verify shapes
        assert output_torch.shape == (seq_len, 4)
        assert output_triton.shape == (seq_len, 4)
        assert output_torch.dtype == torch.uint8
        assert output_triton.dtype == torch.uint8

        # Compare results (should be exact match)
        torch.testing.assert_close(
            output_triton, output_torch, rtol=0, atol=0, msg="GetS outputs differ"
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `torch_fast`, `triton`, `results` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 207-207: Test routines around test_gets_sequential_pages / 测试例程
```python
    def test_gets_sequential_pages(self):
```
**EN:** This range defines concrete test routine(s) `test_gets_sequential_pages`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 208-226: Assertions and result checks / 断言与结果检查
```python
        """Test GetS with sequential page indices."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
        num_pages = 10
        seq_len = 320  # 5 pages

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)

        # Sequential page indices [0, 1, 2, 3, 4]
        page_indices = torch.arange(5, dtype=torch.int32, device=device)

        output_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)
        output_triton = GetS.triton(pool, buf, seq_len, page_indices)

        torch.testing.assert_close(output_triton, output_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `device`, `MockNSATokenToKVPool`, `create_test_buffer` and `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 228-232: Test routines around test_gets_repeated_pages / 测试例程
```python
    def test_gets_repeated_pages(self):
        """Test GetS with repeated page indices."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
```
**EN:** This range defines concrete test routine(s) `test_gets_repeated_pages`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 233-247: Assertions and result checks / 断言与结果检查
```python
        num_pages = 5
        seq_len = 192  # 3 pages

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)

        # Repeated page indices [2, 2, 2]
        page_indices = torch.full((3,), 2, dtype=torch.int32, device=device)

        output_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)
        output_triton = GetS.triton(pool, buf, seq_len, page_indices)

        torch.testing.assert_close(output_triton, output_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `MockNSATokenToKVPool`, `create_test_buffer`, `full` and `torch_fast`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 248-250: Scenario logic / 场景逻辑
```python


@pytest.mark.skipif(not torch.cuda.is_available(), reason="CUDA not available")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipif` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 251-257: Class definition for TestGetKAndS / 类定义
```python
class TestGetKAndS:
    """Test cases for GetKAndS.triton() correctness."""

    @pytest.mark.parametrize("num_pages", [1, 2, 4, 8, 16])
    @pytest.mark.parametrize("seq_len", [64, 128, 256, 512, 1024])
    @pytest.mark.parametrize("page_size", [64])
    @pytest.mark.parametrize("index_head_dim", [128])
```
**EN:** This range declares `TestGetKAndS`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `triton` and `parametrize`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 258-275: Test routines around test_get_k_and_s_correctness / 测试例程
```python
    def test_get_k_and_s_correctness(
        self, num_pages, seq_len, page_size, index_head_dim
    ):
        """Test GetKAndS.triton() produces same output as separate torch_fast calls."""
        device = torch.device("cuda")

        # Ensure seq_len doesn't exceed available pages
        max_seq_len = num_pages * page_size
        seq_len = min(seq_len, max_seq_len)
        seq_len_tensor = torch.tensor([seq_len], dtype=torch.int64, device=device)

        # Create mock pool
        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )

        # Create test buffer
        buf = create_test_buffer(
```
**EN:** This range defines concrete test routine(s) `test_get_k_and_s_correctness`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `triton`, `device`, `tensor` and `MockNSATokenToKVPool`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 276-282: Scenario logic / 场景逻辑
```python
            num_pages=num_pages,
            page_size=page_size,
            index_head_dim=index_head_dim,
            device=device,
        )

        # Create page indices
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 283-300: Assertions and result checks / 断言与结果检查
```python
        num_pages_needed = (seq_len + page_size - 1) // page_size
        page_indices = torch.randint(
            0, num_pages, (num_pages_needed,), dtype=torch.int32, device=device
        )
        page_indices_ = page_indices.unsqueeze(0)

        # Run baseline: separate torch_fast calls
        k_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        s_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)

        # Run fused Triton implementation
        k_triton, s_triton = GetKAndS.triton(
            pool, buf, page_indices_, seq_len_tensor, seq_len, seq_len
        )

        # Verify shapes
        assert k_torch.shape == (seq_len, index_head_dim)
        assert s_torch.shape == (seq_len, 4)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `randint`, `unsqueeze`, `torch_fast` and `triton`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 301-307: Assertions and result checks / 断言与结果检查
```python
        assert k_triton.shape == (seq_len, index_head_dim)
        assert s_triton.shape == (seq_len, 4)

        # Verify dtypes
        assert k_torch.dtype == torch.uint8
        assert s_torch.dtype == torch.uint8
        assert k_triton.dtype == torch.uint8
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。

### Lines 308-318: Assertions and result checks / 断言与结果检查
```python
        assert s_triton.dtype == torch.uint8

        # Compare K results
        torch.testing.assert_close(
            k_triton, k_torch, rtol=0, atol=0, msg="GetKAndS K outputs differ"
        )

        # Compare S results
        torch.testing.assert_close(
            s_triton, s_torch, rtol=0, atol=0, msg="GetKAndS S outputs differ"
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 320-325: Test routines around test_get_k_and_s_sequential_pages / 测试例程
```python
    def test_get_k_and_s_sequential_pages(self):
        """Test GetKAndS with sequential page indices."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
        num_pages = 10
```
**EN:** This range defines concrete test routine(s) `test_get_k_and_s_sequential_pages`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 326-344: Scenario logic / 场景逻辑
```python
        seq_len = 320  # 5 pages
        seq_len_tensor = torch.tensor([seq_len], dtype=torch.int64, device=device)

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)

        # Sequential page indices [0, 1, 2, 3, 4]
        page_indices = torch.arange(5, dtype=torch.int32, device=device)
        page_indices_ = page_indices.unsqueeze(0)

        # Baseline
        k_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        s_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)

        # Fused
        k_triton, s_triton = GetKAndS.triton(
            pool, buf, page_indices_, seq_len_tensor, seq_len, seq_len
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`, `MockNSATokenToKVPool`, `create_test_buffer` and `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 345-348: Assertions and result checks / 断言与结果检查
```python
        )

        torch.testing.assert_close(k_triton, k_torch, rtol=0, atol=0)
        torch.testing.assert_close(s_triton, s_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 350-350: Test routines around test_get_k_and_s_repeated_pages / 测试例程
```python
    def test_get_k_and_s_repeated_pages(self):
```
**EN:** This range defines concrete test routine(s) `test_get_k_and_s_repeated_pages`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 351-374: Scenario logic / 场景逻辑
```python
        """Test GetKAndS with repeated page indices."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
        num_pages = 5
        seq_len = 192  # 3 pages
        seq_len_tensor = torch.tensor([seq_len], dtype=torch.int64, device=device)

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)

        # Repeated page indices [2, 2, 2]
        page_indices = torch.full((3,), 2, dtype=torch.int32, device=device)
        page_indices_ = page_indices.unsqueeze(0)

        # Baseline
        k_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        s_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)

        # Fused
        k_triton, s_triton = GetKAndS.triton(
            pool, buf, page_indices_, seq_len_tensor, seq_len, seq_len
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `device`, `tensor`, `MockNSATokenToKVPool` and `create_test_buffer`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 375-375: Scenario logic / 场景逻辑
```python
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 376-378: Assertions and result checks / 断言与结果检查
```python

        torch.testing.assert_close(k_triton, k_torch, rtol=0, atol=0)
        torch.testing.assert_close(s_triton, s_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 380-400: Test routines around test_get_k_and_s_partial_page / 测试例程
```python
    def test_get_k_and_s_partial_page(self):
        """Test GetKAndS when seq_len is not a multiple of page_size."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
        num_pages = 5
        seq_len = 100  # Not a multiple of 64
        seq_len_tensor = torch.tensor([seq_len], dtype=torch.int64, device=device)

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)

        num_pages_needed = (seq_len + page_size - 1) // page_size
        page_indices = torch.arange(num_pages_needed, dtype=torch.int32, device=device)
        page_indices_ = page_indices.unsqueeze(0)

        # Baseline
        k_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        s_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)
```
**EN:** This range defines concrete test routine(s) `test_get_k_and_s_partial_page`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device`, `tensor`, `MockNSATokenToKVPool` and `create_test_buffer`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 401-404: Scenario logic / 场景逻辑
```python

        # Fused
        k_triton, s_triton = GetKAndS.triton(
            pool, buf, page_indices_, seq_len_tensor, seq_len, seq_len
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `triton`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 405-409: Assertions and result checks / 断言与结果检查
```python
        )

        # Should handle partial pages correctly
        torch.testing.assert_close(k_triton, k_torch, rtol=0, atol=0)
        torch.testing.assert_close(s_triton, s_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 410-412: Scenario logic / 场景逻辑
```python


@pytest.mark.skipif(not torch.cuda.is_available(), reason="CUDA not available")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipif` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 413-415: Class definition for TestEdgeCases / 类定义
```python
class TestEdgeCases:
    """Test edge cases and boundary conditions."""
```
**EN:** This range declares `TestEdgeCases`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 416-437: Test routines around test_single_token / 测试例程
```python
    def test_single_token(self):
        """Test with seq_len=1 (single token)."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
        num_pages = 2
        seq_len = 1
        seq_len_tensor = torch.tensor([seq_len], dtype=torch.int64, device=device)

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)
        page_indices = torch.tensor([0], dtype=torch.int32, device=device)
        page_indices_ = page_indices.unsqueeze(0)

        # Test GetK
        k_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        k_triton = GetK.triton(pool, buf, seq_len, page_indices)
        torch.testing.assert_close(k_triton, k_torch, rtol=0, atol=0)

        # Test GetS
```
**EN:** This range defines concrete test routine(s) `test_single_token`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `device`, `tensor`, `MockNSATokenToKVPool` and `create_test_buffer`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 438-440: Assertions and result checks / 断言与结果检查
```python
        s_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)
        s_triton = GetS.triton(pool, buf, seq_len, page_indices)
        torch.testing.assert_close(s_triton, s_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `torch_fast`, `triton` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 441-447: Assertions and result checks / 断言与结果检查
```python

        # Test GetKAndS
        k_triton2, s_triton2 = GetKAndS.triton(
            pool, buf, page_indices_, seq_len_tensor, seq_len, seq_len
        )
        torch.testing.assert_close(k_triton2, k_torch, rtol=0, atol=0)
        torch.testing.assert_close(s_triton2, s_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `triton` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 449-462: Test routines around test_exact_page_boundary / 测试例程
```python
    def test_exact_page_boundary(self):
        """Test when seq_len exactly matches page boundaries."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
        num_pages = 5
        seq_len = 192  # Exactly 3 pages
        seq_len_tensor = torch.tensor([seq_len], dtype=torch.int64, device=device)

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)
        page_indices = torch.arange(3, dtype=torch.int32, device=device)
```
**EN:** This range defines concrete test routine(s) `test_exact_page_boundary`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device`, `tensor`, `MockNSATokenToKVPool` and `create_test_buffer`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 463-473: Assertions and result checks / 断言与结果检查
```python
        page_indices_ = page_indices.unsqueeze(0)

        # Test GetK
        k_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        k_triton = GetK.triton(pool, buf, seq_len, page_indices)
        torch.testing.assert_close(k_triton, k_torch, rtol=0, atol=0)

        # Test GetS
        s_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)
        s_triton = GetS.triton(pool, buf, seq_len, page_indices)
        torch.testing.assert_close(s_triton, s_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `unsqueeze`, `torch_fast`, `triton` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 474-480: Assertions and result checks / 断言与结果检查
```python

        # Test GetKAndS
        k_triton2, s_triton2 = GetKAndS.triton(
            pool, buf, page_indices_, seq_len_tensor, seq_len, seq_len
        )
        torch.testing.assert_close(k_triton2, k_torch, rtol=0, atol=0)
        torch.testing.assert_close(s_triton2, s_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `triton` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 482-487: Test routines around test_large_seq_len / 测试例程
```python
    def test_large_seq_len(self):
        """Test with large sequence length."""
        device = torch.device("cuda")
        page_size = 64
        index_head_dim = 128
        num_pages = 100
```
**EN:** This range defines concrete test routine(s) `test_large_seq_len`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `device`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 488-506: Assertions and result checks / 断言与结果检查
```python
        seq_len = 4096  # 64 pages
        seq_len_tensor = torch.tensor([seq_len], dtype=torch.int64, device=device)

        pool = MockNSATokenToKVPool(
            page_size=page_size, index_head_dim=index_head_dim, device=device
        )
        buf = create_test_buffer(num_pages, page_size, index_head_dim, device)

        num_pages_needed = (seq_len + page_size - 1) // page_size
        page_indices = torch.randint(
            0, num_pages, (num_pages_needed,), dtype=torch.int32, device=device
        )
        page_indices_ = page_indices.unsqueeze(0)

        # Test GetK
        k_torch = GetK.torch_fast(pool, buf, seq_len, page_indices)
        k_triton = GetK.triton(pool, buf, seq_len, page_indices)
        torch.testing.assert_close(k_triton, k_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tensor`, `MockNSATokenToKVPool`, `create_test_buffer` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 507-512: Assertions and result checks / 断言与结果检查
```python
        # Test GetS
        s_torch = GetS.torch_fast(pool, buf, seq_len, page_indices)
        s_triton = GetS.triton(pool, buf, seq_len, page_indices)
        torch.testing.assert_close(s_triton, s_torch, rtol=0, atol=0)

        # Test GetKAndS
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `torch_fast`, `triton` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 513-517: Assertions and result checks / 断言与结果检查
```python
        k_triton2, s_triton2 = GetKAndS.triton(
            pool, buf, page_indices_, seq_len_tensor, seq_len, seq_len
        )
        torch.testing.assert_close(k_triton2, k_torch, rtol=0, atol=0)
        torch.testing.assert_close(s_triton2, s_torch, rtol=0, atol=0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `triton` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 520-530: Helper routines around print_test_summary / 辅助例程
```python
def print_test_summary():
    """Print a summary message about the test suite."""
    print("\n" + "=" * 80)
    print("NSA Indexer K/S Buffer Accessor Correctness Tests")
    print("=" * 80)
    print("Testing Triton implementations against torch_fast baseline:")
    print("  - GetK.triton() vs GetK.torch_fast()")
    print("  - GetS.triton() vs GetS.torch_fast()")
    print("  - GetKAndS.triton() vs separate GetK/GetS torch_fast() calls")
    print("=" * 80)
    print()
```
**EN:** This range implements helper routine(s) `print_test_summary` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `triton` and `torch_fast`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 531-591: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    # Run tests manually
    if not torch.cuda.is_available():
        print("CUDA not available. Skipping tests.")
        exit(0)

    print_test_summary()

    # Run a few sample tests
    print("Running sample correctness tests...\n")

    # Test GetK
    print("Testing GetK...")
    test_getk = TestGetK()
    test_getk.test_getk_correctness(
        num_pages=4, seq_len=256, page_size=64, index_head_dim=128
    )
    test_getk.test_getk_sequential_pages()
    print("✓ GetK tests passed\n")

    # Test GetS
    print("Testing GetS...")
    test_gets = TestGetS()
    test_gets.test_gets_correctness(
        num_pages=4, seq_len=256, page_size=64, index_head_dim=128
    )
    test_gets.test_gets_sequential_pages()
    print("✓ GetS tests passed\n")

    # Test GetKAndS
    print("Testing GetKAndS SeqLen=256...")
    test_get_k_and_s = TestGetKAndS()
    test_get_k_and_s.test_get_k_and_s_correctness(
        num_pages=4, seq_len=256, page_size=64, index_head_dim=128
    )
    test_get_k_and_s.test_get_k_and_s_sequential_pages()
    test_get_k_and_s.test_get_k_and_s_partial_page()
    print("✓ GetKAndS SeqLen=256 tests passed\n")

    print("Testing GetKAndS SeqLen=128K...")
    test_get_k_and_s = TestGetKAndS()
    test_get_k_and_s.test_get_k_and_s_correctness(
        num_pages=2048, seq_len=131072, page_size=64, index_head_dim=128
    )
    test_get_k_and_s.test_get_k_and_s_sequential_pages()
    test_get_k_and_s.test_get_k_and_s_partial_page()
    print("✓ GetKAndS SeqLen=128K tests passed\n")

    # Test edge cases
    print("Testing edge cases...")
    test_edge = TestEdgeCases()
    test_edge.test_single_token()
    test_edge.test_exact_page_boundary()
    test_edge.test_large_seq_len()
    print("✓ Edge case tests passed\n")

    print("=" * 80)
    print("All correctness tests passed successfully!")
    print("=" * 80)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `is_available`, `exit`, `print_test_summary` and `TestGetK`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Pytest markers / Pytest 标记
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: None / 无
- **Third-party / 第三方库**: `pytest`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.attention.nsa.index_buf_accessor`
