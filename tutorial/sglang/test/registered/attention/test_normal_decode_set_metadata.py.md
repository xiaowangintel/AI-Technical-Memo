# test_normal_decode_set_metadata.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/attention/test_normal_decode_set_metadata.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on attention normal decode set metadata in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 attention normal decode set metadata 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Document the module
```python
"""
Unit tests for the fused Triton kernel in normal_decode_set_metadata.

This test suite verifies:
1. Correctness against reference PyTorch implementation
2. Different page sizes (1, 16, 64)
3. With and without Sliding Window Attention (SWA)
4. Various batch sizes and sequence lengths
5. Edge cases
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 12-21: Import dependencies
```python
import unittest

import torch

from sglang.srt.layers.attention.flashattention_backend import (
    normal_decode_set_metadata,
)
from sglang.srt.mem_cache.swa_memory_pool import SWAKVPool
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 24-24: Register CI metadata
```python
register_cuda_ci(est_time=11, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 27-55: Define helper: reference normal decode set metadata
```python
def reference_normal_decode_set_metadata(
    cache_seqlens_int32: torch.Tensor,
    cu_seqlens_k: torch.Tensor,
    page_table: torch.Tensor,
    req_to_token: torch.Tensor,
    req_pool_indices: torch.Tensor,
    strided_indices: torch.Tensor,
    max_seq_pages: int,
    seq_lens: torch.Tensor,
    seq_len_delta: int,
    page_size: int,
    swa_page_table: torch.Tensor = None,
    token_to_kv_pool=None,
):
    """
    Reference implementation using original PyTorch operations.
    This is the pre-Triton version for correctness comparison.
    """
    cache_seqlens_int32.copy_(seq_lens + seq_len_delta)
    cu_seqlens_k[1:].copy_(torch.cumsum(cache_seqlens_int32, dim=0, dtype=torch.int32))
    page_indices = req_to_token[
        req_pool_indices[:, None],
        strided_indices[:max_seq_pages][None, :],
    ]
    page_table[:, :max_seq_pages].copy_(page_indices // page_size)

    if swa_page_table is not None and token_to_kv_pool is not None:
        swa_page_indices = token_to_kv_pool.translate_loc_from_full_to_swa(page_indices)
        swa_page_table[:, :max_seq_pages].copy_(swa_page_indices // page_size)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 58-59: Define class TestNormalDecodeSetMetadata
```python
@unittest.skipIf(not torch.cuda.is_available(), "Test requires CUDA")
class TestNormalDecodeSetMetadata(CustomTestCase):
```
**EN:** This declaration introduces the `TestNormalDecodeSetMetadata` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNormalDecodeSetMetadata` 测试类，并说明它通过继承承担的职责。

### Lines 60-60: Document the class `TestNormalDecodeSetMetadata`
```python
    """Test fused Triton kernel in normal_decode_set_metadata."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNormalDecodeSetMetadata`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNormalDecodeSetMetadata`的设计意图。

### Lines 62-64: Prepare per-test state
```python
    def setUp(self):
        self.device = "cuda"
        self.dtype = torch.int32
```
**EN:** This setup method initializes the state required before each individual test executes.
**CN:** 该初始化方法会在每个独立测试执行前准备所需状态。

### Lines 66-119: Define helper: create test data (part 1)
```python
    def _create_test_data(
        self,
        batch_size: int,
        max_seq_len: int,
        page_size: int,
        has_swa: bool = False,
        seq_len_delta: int = 0,
    ):
        """Create test data for normal_decode_set_metadata."""
        # Random sequence lengths for each batch
        seq_lens = torch.randint(
            max_seq_len // 2,
            max_seq_len + 1,
            (batch_size,),
            dtype=torch.int64,
            device=self.device,
        )

        # Calculate max_seq_pages
        max_len = seq_lens.max().item()
        max_seq_pages = (max_len + seq_len_delta + page_size - 1) // page_size

        # Create req_pool_indices (maps batch index to pool index)
        req_pool_indices = torch.arange(
            batch_size, dtype=torch.int32, device=self.device
        )

        # Create strided_indices for page table indexing
        if page_size == 1:
            strided_indices = torch.arange(
                max_seq_len * 2, dtype=torch.int32, device=self.device
            )
        else:
            strided_indices = torch.arange(
                0, max_seq_len * 2, page_size, dtype=torch.int32, device=self.device
            )

        # Create req_to_token pool (simulates token locations in KV cache)
        pool_size = batch_size
        max_tokens = max_seq_len * 2
        req_to_token = torch.randint(
            0, 10000, (pool_size, max_tokens), dtype=torch.int32, device=self.device
        )

        # Output tensors (to be filled by the function)
        cache_seqlens_int32 = torch.zeros(
            batch_size, dtype=torch.int32, device=self.device
        )
        cu_seqlens_k = torch.zeros(
            batch_size + 1, dtype=torch.int32, device=self.device
        )
        page_table = torch.zeros(
            (batch_size, max_seq_pages + 10), dtype=torch.int32, device=self.device
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestNormalDecodeSetMetadata` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNormalDecodeSetMetadata` 内部调用，从而让场景结构更清晰。

### Lines 121-129: Define helper: create test data (part 2)
```python
        # SWA setup if needed
        swa_page_table = None
        token_to_kv_pool = None
        if has_swa:
            swa_page_table = torch.zeros(
                (batch_size, max_seq_pages + 10), dtype=torch.int32, device=self.device
            )
            # Create a simple SWA KV pool for testing
            token_to_kv_pool = self._create_swa_kv_pool(10000, page_size)
```
**EN:** This helper function encapsulates reusable logic inside `TestNormalDecodeSetMetadata` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNormalDecodeSetMetadata` 内部调用，从而让场景结构更清晰。

### Lines 131-144: Define helper: create test data (part 3)
```python
        return {
            "cache_seqlens_int32": cache_seqlens_int32,
            "cu_seqlens_k": cu_seqlens_k,
            "page_table": page_table,
            "req_to_token": req_to_token,
            "req_pool_indices": req_pool_indices,
            "strided_indices": strided_indices,
            "max_seq_pages": max_seq_pages,
            "seq_lens": seq_lens,
            "seq_len_delta": seq_len_delta,
            "page_size": page_size,
            "swa_page_table": swa_page_table,
            "token_to_kv_pool": token_to_kv_pool,
        }
```
**EN:** This helper function encapsulates reusable logic inside `TestNormalDecodeSetMetadata` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNormalDecodeSetMetadata` 内部调用，从而让场景结构更清晰。

### Lines 146-168: Define helper: create swa kv pool
```python
    def _create_swa_kv_pool(self, size: int, page_size: int):
        """Create a mock SWA KV pool for testing that inherits from SWAKVPool."""

        # Create a minimal mock that inherits from SWAKVPool to pass isinstance check
        class MinimalSWAKVPool(SWAKVPool):
            def __init__(self, size, device):
                # Don't call super().__init__() to avoid complex initialization
                # Just set the minimal attributes needed for the test
                self.full_to_swa_index_mapping = torch.arange(
                    size, dtype=torch.int32, device=device
                )
                # Add some randomness to simulate real SWA mapping
                self.full_to_swa_index_mapping = (
                    self.full_to_swa_index_mapping
                    + torch.randint(0, 100, (size,), device=device)
                ) % size
                self.device = device

            def translate_loc_from_full_to_swa(self, page_indices):
                """Mock translation method."""
                return self.full_to_swa_index_mapping[page_indices]

        return MinimalSWAKVPool(size, self.device)
```
**EN:** This helper function encapsulates reusable logic inside `TestNormalDecodeSetMetadata` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNormalDecodeSetMetadata` 内部调用，从而让场景结构更清晰。

### Lines 170-222: Define helper: run test (part 1)
```python
    def _run_test(
        self,
        batch_size: int,
        max_seq_len: int,
        page_size: int,
        has_swa: bool = False,
        seq_len_delta: int = 0,
    ):
        """Run a single test configuration."""
        # Create test data
        test_data = self._create_test_data(
            batch_size, max_seq_len, page_size, has_swa, seq_len_delta
        )

        # Clone data for reference implementation
        ref_data = {
            "cache_seqlens_int32": test_data["cache_seqlens_int32"].clone(),
            "cu_seqlens_k": test_data["cu_seqlens_k"].clone(),
            "page_table": test_data["page_table"].clone(),
            "swa_page_table": test_data["swa_page_table"].clone() if has_swa else None,
        }

        # Run reference implementation
        reference_normal_decode_set_metadata(
            ref_data["cache_seqlens_int32"],
            ref_data["cu_seqlens_k"],
            ref_data["page_table"],
            test_data["req_to_token"],
            test_data["req_pool_indices"],
            test_data["strided_indices"],
            test_data["max_seq_pages"],
            test_data["seq_lens"],
            test_data["seq_len_delta"],
            test_data["page_size"],
            ref_data["swa_page_table"],
            test_data["token_to_kv_pool"],
        )

        # Run fused Triton implementation
        normal_decode_set_metadata(
            test_data["cache_seqlens_int32"],
            test_data["cu_seqlens_k"],
            test_data["page_table"],
            test_data["req_to_token"],
            test_data["req_pool_indices"],
            test_data["strided_indices"],
            test_data["max_seq_pages"],
            test_data["seq_lens"],
            test_data["seq_len_delta"],
            test_data["page_size"],
            test_data["swa_page_table"],
            test_data["token_to_kv_pool"],
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestNormalDecodeSetMetadata` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNormalDecodeSetMetadata` 内部调用，从而让场景结构更清晰。

### Lines 224-240: Define helper: run test (part 2)
```python
        # Compare results
        self.assertTrue(
            torch.equal(
                test_data["cache_seqlens_int32"], ref_data["cache_seqlens_int32"]
            ),
            f"cache_seqlens_int32 mismatch. Expected:\n{ref_data['cache_seqlens_int32']}\nGot:\n{test_data['cache_seqlens_int32']}",
        )

        self.assertTrue(
            torch.equal(test_data["cu_seqlens_k"], ref_data["cu_seqlens_k"]),
            f"cu_seqlens_k mismatch. Expected:\n{ref_data['cu_seqlens_k']}\nGot:\n{test_data['cu_seqlens_k']}",
        )

        self.assertTrue(
            torch.equal(test_data["page_table"], ref_data["page_table"]),
            f"page_table mismatch at bs={batch_size}, page_size={page_size}",
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestNormalDecodeSetMetadata` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNormalDecodeSetMetadata` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 242-246: Define helper: run test (part 3)
```python
        if has_swa:
            self.assertTrue(
                torch.equal(test_data["swa_page_table"], ref_data["swa_page_table"]),
                f"swa_page_table mismatch at bs={batch_size}, page_size={page_size}",
            )
```
**EN:** This helper function encapsulates reusable logic inside `TestNormalDecodeSetMetadata` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNormalDecodeSetMetadata` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 249-251: Run test: page size 1 small batch
```python
    def test_page_size_1_small_batch(self):
        """Test with page_size=1, small batch."""
        self._run_test(batch_size=2, max_seq_len=128, page_size=1, has_swa=False)
```
**EN:** This test method exercises page size 1 small batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 1 small batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 253-255: Run test: page size 1 medium batch
```python
    def test_page_size_1_medium_batch(self):
        """Test with page_size=1, medium batch."""
        self._run_test(batch_size=16, max_seq_len=256, page_size=1, has_swa=False)
```
**EN:** This test method exercises page size 1 medium batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 1 medium batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 257-259: Run test: page size 1 large batch
```python
    def test_page_size_1_large_batch(self):
        """Test with page_size=1, large batch."""
        self._run_test(batch_size=64, max_seq_len=512, page_size=1, has_swa=False)
```
**EN:** This test method exercises page size 1 large batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 1 large batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 261-265: Run test: page size 1 with seq len delta
```python
    def test_page_size_1_with_seq_len_delta(self):
        """Test with page_size=1 and seq_len_delta > 0."""
        self._run_test(
            batch_size=8, max_seq_len=200, page_size=1, has_swa=False, seq_len_delta=5
        )
```
**EN:** This test method exercises page size 1 with seq len delta and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 1 with seq len delta 场景，并验证观测到的行为是否符合预期契约。

### Lines 268-270: Run test: page size 16 small batch
```python
    def test_page_size_16_small_batch(self):
        """Test with page_size=16, small batch."""
        self._run_test(batch_size=4, max_seq_len=256, page_size=16, has_swa=False)
```
**EN:** This test method exercises page size 16 small batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 16 small batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 272-274: Run test: page size 16 medium batch
```python
    def test_page_size_16_medium_batch(self):
        """Test with page_size=16, medium batch."""
        self._run_test(batch_size=16, max_seq_len=512, page_size=16, has_swa=False)
```
**EN:** This test method exercises page size 16 medium batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 16 medium batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 276-278: Run test: page size 64 small batch
```python
    def test_page_size_64_small_batch(self):
        """Test with page_size=64, small batch."""
        self._run_test(batch_size=4, max_seq_len=512, page_size=64, has_swa=False)
```
**EN:** This test method exercises page size 64 small batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 64 small batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 280-282: Run test: page size 64 medium batch
```python
    def test_page_size_64_medium_batch(self):
        """Test with page_size=64, medium batch."""
        self._run_test(batch_size=32, max_seq_len=1024, page_size=64, has_swa=False)
```
**EN:** This test method exercises page size 64 medium batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 64 medium batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 284-288: Run test: page size 64 with seq len delta
```python
    def test_page_size_64_with_seq_len_delta(self):
        """Test with page_size=64 and seq_len_delta > 0."""
        self._run_test(
            batch_size=8, max_seq_len=512, page_size=64, has_swa=False, seq_len_delta=3
        )
```
**EN:** This test method exercises page size 64 with seq len delta and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 64 with seq len delta 场景，并验证观测到的行为是否符合预期契约。

### Lines 291-293: Run test: page size 16 with swa
```python
    def test_page_size_16_with_swa(self):
        """Test with page_size=16 and SWA enabled."""
        self._run_test(batch_size=8, max_seq_len=256, page_size=16, has_swa=True)
```
**EN:** This test method exercises page size 16 with swa and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 16 with swa 场景，并验证观测到的行为是否符合预期契约。

### Lines 295-297: Run test: page size 64 with swa
```python
    def test_page_size_64_with_swa(self):
        """Test with page_size=64 and SWA enabled."""
        self._run_test(batch_size=16, max_seq_len=512, page_size=64, has_swa=True)
```
**EN:** This test method exercises page size 64 with swa and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 64 with swa 场景，并验证观测到的行为是否符合预期契约。

### Lines 299-303: Run test: page size 64 with swa and delta
```python
    def test_page_size_64_with_swa_and_delta(self):
        """Test with page_size=64, SWA, and seq_len_delta."""
        self._run_test(
            batch_size=8, max_seq_len=400, page_size=64, has_swa=True, seq_len_delta=2
        )
```
**EN:** This test method exercises page size 64 with swa and delta and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 page size 64 with swa and delta 场景，并验证观测到的行为是否符合预期契约。

### Lines 306-309: Run test: batch size 1
```python
    def test_batch_size_1(self):
        """Test with single batch."""
        self._run_test(batch_size=1, max_seq_len=128, page_size=1, has_swa=False)
        self._run_test(batch_size=1, max_seq_len=256, page_size=64, has_swa=False)
```
**EN:** This test method exercises batch size 1 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 batch size 1 场景，并验证观测到的行为是否符合预期契约。

### Lines 311-338: Run test: max seq pages small
```python
    def test_max_seq_pages_small(self):
        """Test edge case where max_seq_pages could be very small."""
        # This tests when sequences are very short
        test_data = self._create_test_data(
            batch_size=2, max_seq_len=10, page_size=64, has_swa=False
        )

        # Run fused implementation (should handle gracefully)
        normal_decode_set_metadata(
            test_data["cache_seqlens_int32"],
            test_data["cu_seqlens_k"],
            test_data["page_table"],
            test_data["req_to_token"],
            test_data["req_pool_indices"],
            test_data["strided_indices"],
            test_data["max_seq_pages"],
            test_data["seq_lens"],
            test_data["seq_len_delta"],
            test_data["page_size"],
            test_data["swa_page_table"],
            test_data["token_to_kv_pool"],
        )

        # Verify no crashes and basic properties
        self.assertEqual(
            test_data["cache_seqlens_int32"].sum().item(),
            test_data["seq_lens"].sum().item(),
        )
```
**EN:** This test method exercises max seq pages small and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 max seq pages small 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 340-347: Run test: power of two page sizes
```python
    def test_power_of_two_page_sizes(self):
        """Test various power-of-2 page sizes."""
        page_sizes = [1, 2, 4, 8, 16, 32, 64, 128]
        for page_size in page_sizes:
            with self.subTest(page_size=page_size):
                self._run_test(
                    batch_size=4, max_seq_len=256, page_size=page_size, has_swa=False
                )
```
**EN:** This test method exercises power of two page sizes and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 power of two page sizes 场景，并验证观测到的行为是否符合预期契约。

### Lines 349-404: Run test: varied sequence lengths (part 1)
```python
    def test_varied_sequence_lengths(self):
        """Test with highly varied sequence lengths in the same batch."""
        batch_size = 8
        max_seq_len = 512
        page_size = 64

        test_data = self._create_test_data(
            batch_size, max_seq_len, page_size, has_swa=False
        )

        # Manually set varied sequence lengths
        test_data["seq_lens"] = torch.tensor(
            [10, 50, 100, 200, 300, 450, 500, 512],
            dtype=torch.int64,
            device=self.device,
        )
        test_data["max_seq_pages"] = (
            test_data["seq_lens"].max().item() + page_size - 1
        ) // page_size

        # Run both implementations
        ref_data = {
            "cache_seqlens_int32": test_data["cache_seqlens_int32"].clone(),
            "cu_seqlens_k": test_data["cu_seqlens_k"].clone(),
            "page_table": test_data["page_table"].clone(),
        }

        reference_normal_decode_set_metadata(
            ref_data["cache_seqlens_int32"],
            ref_data["cu_seqlens_k"],
            ref_data["page_table"],
            test_data["req_to_token"],
            test_data["req_pool_indices"],
            test_data["strided_indices"],
            test_data["max_seq_pages"],
            test_data["seq_lens"],
            0,
            page_size,
            None,
            None,
        )

        normal_decode_set_metadata(
            test_data["cache_seqlens_int32"],
            test_data["cu_seqlens_k"],
            test_data["page_table"],
            test_data["req_to_token"],
            test_data["req_pool_indices"],
            test_data["strided_indices"],
            test_data["max_seq_pages"],
            test_data["seq_lens"],
            0,
            page_size,
            None,
            None,
        )
```
**EN:** This test method exercises varied sequence lengths and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 varied sequence lengths 场景，并验证观测到的行为是否符合预期契约。

### Lines 406-414: Run test: varied sequence lengths (part 2)
```python
        self.assertTrue(
            torch.equal(
                test_data["cache_seqlens_int32"], ref_data["cache_seqlens_int32"]
            )
        )
        self.assertTrue(
            torch.equal(test_data["cu_seqlens_k"], ref_data["cu_seqlens_k"])
        )
        self.assertTrue(torch.equal(test_data["page_table"], ref_data["page_table"]))
```
**EN:** This test method exercises varied sequence lengths and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 varied sequence lengths 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 417-418: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.layers.attention.flashattention_backend`, `sglang.srt.mem_cache.swa_memory_pool`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `CustomTestCase`, `unittest.main`
