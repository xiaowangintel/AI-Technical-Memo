# test_chunked_sgmv_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_chunked_sgmv_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates chunked sgmv backend behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 chunked sgmv backend 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: module imports and dependencies / 模块导入与依赖
```python
import random
import unittest
from enum import Enum
from typing import List, Optional, Tuple

import torch

from sglang.srt.layers.logits_processor import LogitsMetadata, LogitsProcessor
from sglang.srt.lora.backend.chunked_backend import ChunkedSgmvLoRABackend
from sglang.srt.lora.triton_ops import (
    chunked_embedding_lora_a_forward,
    chunked_sgmv_lora_expand_forward,
    chunked_sgmv_lora_shrink_forward,
)
from sglang.srt.lora.triton_ops.chunked_sgmv_expand import _chunked_lora_expand_kernel
from sglang.srt.lora.triton_ops.chunked_sgmv_shrink import _chunked_lora_shrink_kernel
from sglang.srt.lora.utils import LoRABatchInfo, get_lm_head_pruned_lens
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.lora_utils import (
    reference_embedding_lora_a_shrink,
    reference_sgmv_expand,
    reference_sgmv_shrink,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `random`, `unittest`, `enum`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `random`, `unittest`, `enum`, `typing`。

### Lines 26-28: CI registration and metadata / CI 注册与元数据
```python
CHUNK_SIZE = 16

register_cuda_ci(est_time=60, suite="nightly-1-gpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 31-33: function reset kernel cache / 函数 reset kernel cache
```python
def reset_kernel_cache():
    _chunked_lora_shrink_kernel._clear_cache()
    _chunked_lora_expand_kernel._clear_cache()
```
**EN:** This block implements `reset_kernel_cache` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `reset_kernel_cache`，承担模块行为中的一个聚焦逻辑片段。

### Lines 36-36: class BatchComposition declaration / 类 BatchComposition 声明
```python
class BatchComposition(Enum):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `Enum`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `Enum`。

### Lines 37-40: class-level constants and configuration for `BatchComposition` / 类级常量与配置
```python
    UNIFORM = "uniform"
    MIXED = "mixed"
    SKEWED = "skewed"
    NONE = "_NO_LORA_"
```
**EN:** This block defines shared names such as `UNIFORM`, `MIXED`, `SKEWED`, `NONE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `UNIFORM`, `MIXED`, `SKEWED`, `NONE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 43-43: class BatchMode declaration / 类 BatchMode 声明
```python
class BatchMode(Enum):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `Enum`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `Enum`。

### Lines 44-46: class-level constants and configuration for `BatchMode` / 类级常量与配置
```python
    PREFILL = "prefill"
    DECODE = "decode"
    TARGET_VERIFY = "verify"
```
**EN:** This block defines shared names such as `PREFILL`, `DECODE`, `TARGET_VERIFY`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `PREFILL`, `DECODE`, `TARGET_VERIFY` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 49-51: class TestChunkedSGMV declaration / 类 TestChunkedSGMV 声明
```python
class TestChunkedSGMV(unittest.TestCase):

    # Test configuration constants
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 52-54: class-level constants and configuration for `TestChunkedSGMV` / 类级常量与配置
```python
    RTOL = 1e-3
    ATOL = 1e-3
    DEFAULT_BATCH_SIZE = 8
```
**EN:** This block defines shared names such as `RTOL`, `ATOL`, `DEFAULT_BATCH_SIZE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `RTOL`, `ATOL`, `DEFAULT_BATCH_SIZE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 56-102: method compare shrink outputs / 方法 compare shrink outputs
```python
    def _compare_shrink_outputs(
        self,
        chunked_output: torch.Tensor,
        reference_output: torch.Tensor,
        seq_lengths: List[int],
        lora_assignments: List[int],
        batch_info: LoRABatchInfo,
        num_slices: int,
        test_name: str,
    ):
        """
        Compare only the valid portions of shrink outputs.

        The chunked SGMV shrink kernel only guarantees correctness for
        output[seq_start:seq_end, :rank * num_slices] for each sequence.
        """
        lora_ranks = batch_info.lora_ranks.cpu().numpy()

        token_offset = 0
        for seq_idx, (lora_idx, seq_len) in enumerate(
            zip(lora_assignments, seq_lengths)
        ):
            if seq_len == 0:
                continue

            rank = lora_ranks[lora_idx]

            if rank > 0:
                # Only compare the valid columns for this sequence
                valid_cols = num_slices * rank

                chunked_seq = chunked_output[
                    token_offset : token_offset + seq_len, :valid_cols
                ]
                reference_seq = reference_output[
                    token_offset : token_offset + seq_len, :valid_cols
                ]

                torch.testing.assert_close(
                    chunked_seq,
                    reference_seq,
                    rtol=self.RTOL,
                    atol=self.ATOL,
                    msg=f"Shrink operation failed for {test_name}, sequence {seq_idx} ({lora_idx})",
                )

            token_offset += seq_len
```
**EN:** Compare only the valid portions of shrink outputs. This block implements `_compare_shrink_outputs` and captures one focused piece of the module's behavior.
**CN:** Compare only the valid portions of shrink outputs. 该代码块实现 `_compare_shrink_outputs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 104-127: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up common test parameters"""
        torch.manual_seed(42)
        random.seed(42)

        self.device = torch.device("cuda")
        self.dtype = torch.float16
        self.input_dim = 2560  # Hidden dimension
        self.max_seq_len = 1024
        self.vocab_size = 32000  # Vocabulary size for embedding tests

        # LoRA configurations: name -> (rank, output_q, output_k, output_v)
        self.lora_configs = {
            "lora_A": (8, 4096, 1024, 1024),
            "lora_B": (16, 4096, 1024, 1024),
            "lora_C": (32, 4096, 1024, 1024),
            "_NO_LORA_": (0, 4096, 1024, 1024),
        }

        # QKV slice offsets: 4096 (Q) + 1024 (K) + 1024 (V) = 6144 total
        self.slice_offsets = torch.tensor(
            [0, 4096, 5120, 6144], dtype=torch.int32, device=self.device
        )
        self.max_slice_size = 4096
```
**EN:** Set up common test parameters This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up common test parameters 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 129-142: method generate sequence lengths / 方法 generate sequence lengths
```python
    def generate_sequence_lengths(
        self,
        batch_size: int,
        batch_mode: BatchMode = BatchMode.PREFILL,
        min_len: int = 1,
        max_len: int = None,
    ) -> List[int]:
        """Generate sequence lengths for a batch based on mode"""
        if batch_mode == BatchMode.DECODE:
            return [1] * batch_size
        else:
            if max_len is None:
                max_len = self.max_seq_len
            return [random.randint(min_len, max_len) for _ in range(batch_size)]
```
**EN:** Generate sequence lengths for a batch based on mode This block implements `generate_sequence_lengths` and captures one focused piece of the module's behavior.
**CN:** Generate sequence lengths for a batch based on mode 该代码块实现 `generate_sequence_lengths`，承担模块行为中的一个聚焦逻辑片段。

### Lines 144-176: helper routine create lora weights / 辅助流程 create lora weights
```python
    def create_lora_weights(
        self, lora_name: str, include_missing_k: bool = False
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """Create LoRA A and B weights for given configuration"""
        rank, out_q, out_k, out_v = self.lora_configs[lora_name]

        if rank == 0:
            lora_a = torch.empty(
                0, self.input_dim, dtype=self.dtype, device=self.device
            )
            lora_b = torch.empty(
                out_q + out_k + out_v, 0, dtype=self.dtype, device=self.device
            )
            return lora_a, lora_b

        # Create LoRA A weights (3 slices for QKV)
        lora_a = torch.randn(
            3 * rank, self.input_dim, dtype=self.dtype, device=self.device
        )

        if include_missing_k:
            lora_a[rank : 2 * rank, :] = 0.0

        # Create LoRA B weights (stacked Q, K, V)
        total_output_dim = out_q + out_k + out_v
        lora_b = torch.randn(
            total_output_dim, rank, dtype=self.dtype, device=self.device
        )

        if include_missing_k:
            lora_b[out_q : out_q + out_k, :] = 0.0

        return lora_a, lora_b
```
**EN:** Create LoRA A and B weights for given configuration This helper encapsulates `create_lora_weights` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Create LoRA A and B weights for given configuration 该辅助函数封装了 `create_lora_weights`，以便周围测试复用准备、执行或校验逻辑。

### Lines 178-257: helper routine create batch info (part 1/2) / 辅助流程 create batch info（第 1/2 部分）
```python
    def create_batch_info(
        self,
        lora_names: List[str],
        seq_lengths: List[int],
        lora_assignments: List[Optional[int]],
        batch_mode: BatchMode = BatchMode.PREFILL,
    ) -> LoRABatchInfo:
        """Create LoRABatchInfo using the same logic as chunked backend"""
        lora_ranks = [self.lora_configs[name][0] for name in lora_names]

        def create_mock_batch():
            # Create a minimal mock ForwardBatch for the test
            class MockForwardBatch:
                def __init__(self, batch_size, seq_lengths, device):
                    self.batch_size = batch_size
                    self.extend_seq_lens = torch.tensor(
                        seq_lengths, dtype=torch.int32, device=device
                    )
                    self.extend_seq_lens_cpu = seq_lengths
                    self.forward_mode = MockForwardMode()

            class MockForwardMode:
                def is_extend(self):
                    return batch_mode == BatchMode.PREFILL

                def is_decode(self):
                    return batch_mode == BatchMode.DECODE

                def is_target_verify(self):
                    return batch_mode == BatchMode.TARGET_VERIFY

                def is_prefill(self):
                    return self.is_extend()

            return MockForwardBatch(len(seq_lengths), seq_lengths, self.device)

        mock_batch = create_mock_batch()

        # Use the same functions as chunked backend
        permutation, weights_reordered = ChunkedSgmvLoRABackend._get_permutation(
            lora_assignments, mock_batch
        )

        # Create a minimal backend instance to access _get_segments_info
        mock_server_args = type(
            "ServerArgs", (object,), {"max_lora_chunk_size": "MOCK_NEVER_USED"}
        )
        mock_backend = ChunkedSgmvLoRABackend(
            max_loras_per_batch=8, device=self.device, server_args=mock_server_args
        )
        weight_indices_list, seg_indptr = mock_backend._get_segments_info(
            weights_reordered,
            chunk_size=CHUNK_SIZE,
        )

        scalings = [1.0] * len(lora_names)
        seg_indptr_tensor = seg_indptr.to(self.device)
        weight_indices_tensor = weight_indices_list.to(self.device)
        lora_ranks_tensor = (
            torch.tensor(lora_ranks, dtype=torch.int32, device=self.device)
            if lora_ranks
            else torch.empty(0, dtype=torch.int32, device=self.device)
        )
        scalings_tensor = (
            torch.tensor(scalings, dtype=torch.float32, device=self.device)
            if scalings
            else torch.empty(0, dtype=torch.float32, device=self.device)
        )
        permutation_tensor = permutation.to(
            self.device, dtype=torch.int32
        )  # Convert to int32 for LoRABatchInfo
        seq_lens_tensor = torch.tensor(
            seq_lengths, dtype=torch.int32, device=self.device
        )

        return LoRABatchInfo(
            use_cuda_graph=False,
            bs=len(seq_lengths),
            num_segments=len(weight_indices_list),  # Number of segments, not sequences!
            seg_indptr=seg_indptr_tensor,
```
**EN:** Create LoRABatchInfo using the same logic as chunked backend This helper encapsulates `create_batch_info` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 1 of the same logical block.
**CN:** Create LoRABatchInfo using the same logic as chunked backend 该辅助函数封装了 `create_batch_info`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 1 部分。

### Lines 258-264: helper routine create batch info (part 2/2) / 辅助流程 create batch info（第 2/2 部分）
```python
            weight_indices=weight_indices_tensor,
            lora_ranks=lora_ranks_tensor,
            scalings=scalings_tensor,
            seg_lens=seq_lens_tensor,  # Original sequence lengths for reference
            max_len=CHUNK_SIZE,
            permutation=permutation_tensor,  # Token reordering permutation
        )
```
**EN:** Create LoRABatchInfo using the same logic as chunked backend This helper encapsulates `create_batch_info` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 2 of the same logical block.
**CN:** Create LoRABatchInfo using the same logic as chunked backend 该辅助函数封装了 `create_batch_info`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 2 部分。

### Lines 266-297: method stack lora weights / 方法 stack lora weights
```python
    def stack_lora_weights(
        self, weight_list: List[torch.Tensor], is_lora_a: bool
    ) -> torch.Tensor:
        """Stack LoRA weights from different adapters into a single tensor"""
        if not weight_list:
            return torch.empty(0, 0, 0, dtype=self.dtype, device=self.device)

        first_non_empty = next((w for w in weight_list if w.numel() > 0), None)
        if first_non_empty is None:
            return torch.empty(
                len(weight_list), 0, 0, dtype=self.dtype, device=self.device
            )
        if is_lora_a:
            # LoRA A: (slice_num * rank, input_dim) -> (num_loras, slice_num * max_rank, input_dim)
            max_rank = max(w.shape[0] // 3 if w.numel() > 0 else 0 for w in weight_list)
            final_shape = (len(weight_list), 3 * max_rank, self.input_dim)
        else:
            # LoRA B: (output_dim, rank) -> (num_loras, output_dim, max_rank)
            max_rank = max(w.shape[1] if w.numel() > 0 else 0 for w in weight_list)
            output_dim = first_non_empty.shape[0]
            final_shape = (len(weight_list), output_dim, max_rank)

        stacked = torch.zeros(final_shape, dtype=self.dtype, device=self.device)

        for i, weight in enumerate(weight_list):
            if weight.numel() > 0:
                if is_lora_a:
                    stacked[i, : weight.shape[0], :] = weight
                else:
                    stacked[i, :, : weight.shape[1]] = weight

        return stacked
```
**EN:** Stack LoRA weights from different adapters into a single tensor This block implements `stack_lora_weights` and captures one focused piece of the module's behavior.
**CN:** Stack LoRA weights from different adapters into a single tensor 该代码块实现 `stack_lora_weights`，承担模块行为中的一个聚焦逻辑片段。

### Lines 299-327: helper routine create embedding lora a weights / 辅助流程 create embedding lora a weights
```python
    def create_embedding_lora_a_weights(self, lora_ranks: torch.Tensor) -> torch.Tensor:
        """Create LoRA A weights for embedding lookup.

        Args:
            lora_ranks: Tensor of ranks for each LoRA adapter

        Returns:
            Tensor of shape (num_loras, max_rank, vocab_size)
        """
        lora_ranks_cpu = lora_ranks.cpu().numpy()
        num_loras = len(lora_ranks_cpu)
        max_rank = int(lora_ranks_cpu.max()) if num_loras > 0 else 0

        if max_rank == 0:
            return torch.empty(
                num_loras, 0, self.vocab_size, dtype=self.dtype, device=self.device
            )

        weights = torch.zeros(
            num_loras, max_rank, self.vocab_size, dtype=self.dtype, device=self.device
        )

        for i, rank in enumerate(lora_ranks_cpu):
            if rank > 0:
                weights[i, :rank, :] = torch.randn(
                    rank, self.vocab_size, dtype=self.dtype, device=self.device
                )

        return weights
```
**EN:** Create LoRA A weights for embedding lookup. This helper encapsulates `create_embedding_lora_a_weights` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Create LoRA A weights for embedding lookup. 该辅助函数封装了 `create_embedding_lora_a_weights`，以便周围测试复用准备、执行或校验逻辑。

### Lines 329-333: helper routine create test input ids / 辅助流程 create test input ids
```python
    def create_test_input_ids(self, total_tokens: int) -> torch.Tensor:
        """Create random token IDs for embedding test."""
        return torch.randint(
            0, self.vocab_size, (total_tokens,), dtype=torch.int64, device=self.device
        )
```
**EN:** Create random token IDs for embedding test. This helper encapsulates `create_test_input_ids` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Create random token IDs for embedding test. 该辅助函数封装了 `create_test_input_ids`，以便周围测试复用准备、执行或校验逻辑。

### Lines 335-391: helper routine create test batch / 辅助流程 create test batch
```python
    def create_test_batch(
        self,
        batch_composition: BatchComposition,
        batch_size: int,
        batch_mode: BatchMode = BatchMode.PREFILL,
        include_missing_k: bool = False,
    ) -> Tuple[
        torch.Tensor,
        List[Tuple[torch.Tensor, torch.Tensor]],
        LoRABatchInfo,
        List[int],
        List[str],
    ]:
        """Create test batch with specified composition and mode"""

        # Reset kernel cache to avoid cross-test contamination
        reset_kernel_cache()

        seq_lengths = self.generate_sequence_lengths(
            batch_size, batch_mode, 1, self.max_seq_len
        )
        if batch_composition == BatchComposition.UNIFORM:
            lora_names = ["lora_A"]
            lora_assignments = [lora_names.index("lora_A")] * batch_size
        elif batch_composition == BatchComposition.MIXED:
            lora_names = ["lora_A", "lora_B", "lora_C", None]
            lora_assignments = [(i % len(lora_names)) for i in range(batch_size)]
        elif batch_composition == BatchComposition.SKEWED:
            lora_names = ["lora_A", "lora_B"]
            num_minority = max(1, batch_size // 8)
            lora_assignments = [lora_names.index("lora_A")] * num_minority + [
                lora_names.index("lora_B")
            ] * (batch_size - num_minority)
            random.shuffle(lora_assignments)
        elif batch_composition == BatchComposition.NONE:
            lora_names = [None]
            lora_assignments = [0] * batch_size
        else:
            raise ValueError(f"Unknown batch composition: {batch_composition}")

        total_seq_len = sum(seq_lengths)
        x = torch.randn(
            total_seq_len, self.input_dim, dtype=self.dtype, device=self.device
        )

        normalized_lora_names = [
            "_NO_LORA_" if name is None else name for name in lora_names
        ]
        weights = []
        for lora_name in normalized_lora_names:
            weights.append(self.create_lora_weights(lora_name, include_missing_k))

        batch_info = self.create_batch_info(
            normalized_lora_names, seq_lengths, lora_assignments, batch_mode
        )

        return x, weights, batch_info, seq_lengths, lora_assignments
```
**EN:** Create test batch with specified composition and mode This helper encapsulates `create_test_batch` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Create test batch with specified composition and mode 该辅助函数封装了 `create_test_batch`，以便周围测试复用准备、执行或校验逻辑。

### Lines 393-470: helper routine run test comparison / 辅助流程 run test comparison
```python
    def run_test_comparison(
        self,
        x: torch.Tensor,
        weights: List[Tuple[torch.Tensor, torch.Tensor]],
        batch_info: LoRABatchInfo,
        seq_lengths: List[int],
        lora_assignments: List[int],
        test_name: str,
    ):
        """Run comparison between chunked and reference implementations"""
        if not weights:  # Handle case with no LoRA weights
            return

        lora_assignments_tensor = torch.tensor(
            lora_assignments, dtype=torch.int32, device="cpu"
        )
        seq_lengths_tensor = torch.tensor(seq_lengths, dtype=torch.int32, device="cpu")
        lora_ranks_tensor = batch_info.lora_ranks.detach().cpu()
        scalings_tensor = batch_info.scalings.detach().cpu()

        # Stack LoRA A weights
        lora_a_weights = [weight[0] for weight in weights]
        stacked_lora_a = self.stack_lora_weights(lora_a_weights, is_lora_a=True)

        # Stack LoRA B weights
        lora_b_weights = [weight[1] for weight in weights]
        stacked_lora_b = self.stack_lora_weights(lora_b_weights, is_lora_a=False)

        # Test shrink operation
        chunked_shrink = chunked_sgmv_lora_shrink_forward(
            x, stacked_lora_a, batch_info, num_slices=3
        )
        reference_shrink = reference_sgmv_shrink(
            x,
            stacked_lora_a,
            lora_assignments_tensor,
            seq_lengths_tensor,
            lora_ranks_tensor,
            scalings_tensor,
            num_slices=3,
        )

        # Only compare valid portions of shrink output (first rank * num_slices columns per sequence)
        self._compare_shrink_outputs(
            chunked_shrink,
            reference_shrink,
            seq_lengths,
            lora_assignments,
            batch_info,
            num_slices=3,
            test_name=test_name,
        )

        # Test expand operation
        chunked_expand = chunked_sgmv_lora_expand_forward(
            reference_shrink,
            stacked_lora_b,
            batch_info,
            self.slice_offsets,
            self.max_slice_size,
            base_output=None,
        )
        reference_expand = reference_sgmv_expand(
            reference_shrink,
            stacked_lora_b,
            lora_assignments_tensor,
            seq_lengths_tensor,
            lora_ranks_tensor,
            self.slice_offsets,
        )

        torch.testing.assert_close(
            chunked_expand,
            reference_expand,
            rtol=self.RTOL,
            atol=self.ATOL,
            msg=f"Expand operation failed for {test_name}",
        )
```
**EN:** Run comparison between chunked and reference implementations This helper encapsulates `run_test_comparison` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Run comparison between chunked and reference implementations 该辅助函数封装了 `run_test_comparison`，以便周围测试复用准备、执行或校验逻辑。

### Lines 471-473: supporting source context / 辅助源码上下文
```python

    # === Basic Operations Tests ===

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 474-540: test case shrink basic / 测试用例 shrink basic
```python
    def test_shrink_basic(self):
        """Test basic shrink operation against PyTorch reference"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(BatchComposition.UNIFORM, batch_size)
                )

                lora_assignments_tensor = torch.tensor(
                    lora_assignments, dtype=torch.int32, device="cpu"
                )
                seq_lengths_tensor = torch.tensor(
                    seq_lengths, dtype=torch.int32, device="cpu"
                )
                lora_ranks_tensor = batch_info.lora_ranks.detach().cpu()
                scalings_tensor = batch_info.scalings.detach().cpu()

                lora_a_weights = [weight[0] for weight in weights]
                stacked_lora_a = self.stack_lora_weights(lora_a_weights, is_lora_a=True)

                chunked_shrink = chunked_sgmv_lora_shrink_forward(
                    x, stacked_lora_a, batch_info, num_slices=3
                )
                reference_shrink = reference_sgmv_shrink(
                    x,
                    stacked_lora_a,
                    lora_assignments_tensor,
                    seq_lengths_tensor,
                    lora_ranks_tensor,
                    scalings_tensor,
                    num_slices=3,
                )

                torch.testing.assert_close(
                    chunked_shrink, reference_shrink, rtol=self.RTOL, atol=self.ATOL
                )

                # Test chunked embedding LoRA A forward
                # Create embedding-specific LoRA A weights with shape (num_loras, rank, vocab_size)
                embedding_lora_a = self.create_embedding_lora_a_weights(
                    batch_info.lora_ranks
                )

                # Create input_ids (token indices) instead of hidden states
                total_tokens = x.shape[0]
                input_ids = self.create_test_input_ids(total_tokens)

                chunked_shrink_embeddings = chunked_embedding_lora_a_forward(
                    input_ids, embedding_lora_a, batch_info, self.vocab_size
                )

                reference_shrink_embeddings = reference_embedding_lora_a_shrink(
                    input_ids,
                    embedding_lora_a,
                    lora_assignments_tensor,
                    seq_lengths_tensor,
                    lora_ranks_tensor,
                    scalings_tensor,
                    self.vocab_size,
                )
                torch.testing.assert_close(
                    chunked_shrink_embeddings,
                    reference_shrink_embeddings,
                    rtol=self.RTOL,
                    atol=self.ATOL,
                    msg=f"Shrink test embedding loRA A operation failed for batch_size={batch_size}",
                )
```
**EN:** Test basic shrink operation against PyTorch reference This test exercises `test_shrink_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic shrink operation against PyTorch reference 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_shrink_basic`。

### Lines 542-596: test case expand basic / 测试用例 expand basic
```python
    def test_expand_basic(self):
        """Test basic expand operation against PyTorch reference"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(BatchComposition.UNIFORM, batch_size)
                )

                lora_assignments_tensor = torch.tensor(
                    lora_assignments, dtype=torch.int32, device="cpu"
                )
                seq_lengths_tensor = torch.tensor(
                    seq_lengths, dtype=torch.int32, device="cpu"
                )
                lora_ranks_tensor = batch_info.lora_ranks.detach().cpu()
                scalings_tensor = batch_info.scalings.detach().cpu()

                lora_a_weights = [weight[0] for weight in weights]
                stacked_lora_a = self.stack_lora_weights(lora_a_weights, is_lora_a=True)

                intermediate = reference_sgmv_shrink(
                    x,
                    stacked_lora_a,
                    lora_assignments_tensor,
                    seq_lengths_tensor,
                    lora_ranks_tensor,
                    scalings_tensor,
                    num_slices=3,
                )

                lora_b_weights = [weight[1] for weight in weights]
                stacked_lora_b = self.stack_lora_weights(
                    lora_b_weights, is_lora_a=False
                )

                chunked_expand = chunked_sgmv_lora_expand_forward(
                    intermediate,
                    stacked_lora_b,
                    batch_info,
                    self.slice_offsets,
                    self.max_slice_size,
                    base_output=None,
                )
                reference_expand = reference_sgmv_expand(
                    intermediate,
                    stacked_lora_b,
                    lora_assignments_tensor,
                    seq_lengths_tensor,
                    lora_ranks_tensor,
                    self.slice_offsets,
                )

                torch.testing.assert_close(
                    chunked_expand, reference_expand, rtol=self.RTOL, atol=self.ATOL
                )
```
**EN:** Test basic expand operation against PyTorch reference This test exercises `test_expand_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic expand operation against PyTorch reference 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_expand_basic`。

### Lines 597-599: supporting source context / 辅助源码上下文
```python

    # === QKV Operations Test ===

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 600-616: test case qkv missing projections / 测试用例 qkv missing projections
```python
    def test_qkv_missing_projections(self):
        """Test QKV operations with missing k_proj (Qwen3 scenario)"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(
                        BatchComposition.MIXED, batch_size, include_missing_k=True
                    )
                )
                self.run_test_comparison(
                    x,
                    weights,
                    batch_info,
                    seq_lengths,
                    lora_assignments,
                    f"QKV missing k_proj batch_size={batch_size}",
                )
```
**EN:** Test QKV operations with missing k_proj (Qwen3 scenario) This test exercises `test_qkv_missing_projections` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test QKV operations with missing k_proj (Qwen3 scenario) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qkv_missing_projections`。

### Lines 618-697: test case 4 slice gdn qkvz (part 1/2) / 测试用例 4 slice gdn qkvz（第 1/2 部分）
```python
    def test_4_slice_gdn_qkvz(self):
        """Test 4-slice shrink+expand operations (GDN in_proj_qkvz)."""
        num_slices = 4
        # GDN-style: 4 slices with different sizes [2048, 2048, 4096, 4096]
        slice_offsets = torch.tensor(
            [0, 2048, 4096, 8192, 12288], dtype=torch.int32, device=self.device
        )
        total_out = 12288
        max_slice_size = 4096

        for batch_size in [1, 2, 16]:
            with self.subTest(batch_size=batch_size):
                # Build batch (reuse the 3-slice helper just for x, batch_info, etc.)
                x, _, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(BatchComposition.MIXED, batch_size)
                )

                # Build 4-slice LoRA weights and stack them manually
                lora_names = [n for n in self.lora_configs if n != "_NO_LORA_"]
                max_rank = max(self.lora_configs[n][0] for n in lora_names)
                stacked_a = torch.zeros(
                    len(lora_names),
                    num_slices * max_rank,
                    self.input_dim,
                    dtype=self.dtype,
                    device=self.device,
                )
                stacked_b = torch.zeros(
                    len(lora_names),
                    total_out,
                    max_rank,
                    dtype=self.dtype,
                    device=self.device,
                )
                for i, name in enumerate(lora_names):
                    rank = self.lora_configs[name][0]
                    if rank > 0:
                        stacked_a[i, : num_slices * rank, :] = torch.randn(
                            num_slices * rank,
                            self.input_dim,
                            dtype=self.dtype,
                            device=self.device,
                        )
                        stacked_b[i, :, :rank] = torch.randn(
                            total_out, rank, dtype=self.dtype, device=self.device
                        )

                lora_assignments_tensor = torch.tensor(
                    lora_assignments, dtype=torch.int32, device="cpu"
                )
                seq_lengths_tensor = torch.tensor(
                    seq_lengths, dtype=torch.int32, device="cpu"
                )
                lora_ranks_tensor = batch_info.lora_ranks.detach().cpu()
                scalings_tensor = batch_info.scalings.detach().cpu()

                # Shrink
                chunked_shrink = chunked_sgmv_lora_shrink_forward(
                    x, stacked_a, batch_info, num_slices=num_slices
                )
                reference_shrink = reference_sgmv_shrink(
                    x,
                    stacked_a,
                    lora_assignments_tensor,
                    seq_lengths_tensor,
                    lora_ranks_tensor,
                    scalings_tensor,
                    num_slices=num_slices,
                )
                self._compare_shrink_outputs(
                    chunked_shrink,
                    reference_shrink,
                    seq_lengths,
                    lora_assignments,
                    batch_info,
                    num_slices=num_slices,
                    test_name=f"4-slice shrink bs={batch_size}",
                )

                # Expand
```
**EN:** Test 4-slice shrink+expand operations (GDN in_proj_qkvz). This test exercises `test_4_slice_gdn_qkvz` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test 4-slice shrink+expand operations (GDN in_proj_qkvz). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_4_slice_gdn_qkvz`。 这一段对应同一逻辑块的第 1 部分。

### Lines 698-720: test case 4 slice gdn qkvz (part 2/2) / 测试用例 4 slice gdn qkvz（第 2/2 部分）
```python
                chunked_expand = chunked_sgmv_lora_expand_forward(
                    reference_shrink,
                    stacked_b,
                    batch_info,
                    slice_offsets,
                    max_slice_size,
                    base_output=None,
                )
                reference_expand = reference_sgmv_expand(
                    reference_shrink,
                    stacked_b,
                    lora_assignments_tensor,
                    seq_lengths_tensor,
                    lora_ranks_tensor,
                    slice_offsets,
                )
                torch.testing.assert_close(
                    chunked_expand,
                    reference_expand,
                    rtol=self.RTOL,
                    atol=self.ATOL,
                    msg=f"4-slice expand failed bs={batch_size}",
                )
```
**EN:** Test 4-slice shrink+expand operations (GDN in_proj_qkvz). This test exercises `test_4_slice_gdn_qkvz` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test 4-slice shrink+expand operations (GDN in_proj_qkvz). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_4_slice_gdn_qkvz`。 这一段对应同一逻辑块的第 2 部分。

### Lines 721-723: supporting source context / 辅助源码上下文
```python

    # === Batch Composition Tests ===

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 724-738: test case uniform lora batch / 测试用例 uniform lora batch
```python
    def test_uniform_lora_batch(self):
        """All sequences use same LoRA, random sequence lengths"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(BatchComposition.UNIFORM, batch_size)
                )
                self.run_test_comparison(
                    x,
                    weights,
                    batch_info,
                    seq_lengths,
                    lora_assignments,
                    f"uniform batch_size={batch_size}",
                )
```
**EN:** All sequences use same LoRA, random sequence lengths This test exercises `test_uniform_lora_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** All sequences use same LoRA, random sequence lengths 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_uniform_lora_batch`。

### Lines 740-754: test case evenly mixed lora batch / 测试用例 evenly mixed lora batch
```python
    def test_evenly_mixed_lora_batch(self):
        """Sequences evenly distributed across LoRAs, random lengths"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(BatchComposition.MIXED, batch_size)
                )
                self.run_test_comparison(
                    x,
                    weights,
                    batch_info,
                    seq_lengths,
                    lora_assignments,
                    f"mixed batch_size={batch_size}",
                )
```
**EN:** Sequences evenly distributed across LoRAs, random lengths This test exercises `test_evenly_mixed_lora_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Sequences evenly distributed across LoRAs, random lengths 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_evenly_mixed_lora_batch`。

### Lines 756-770: test case highly skewed lora batch / 测试用例 highly skewed lora batch
```python
    def test_highly_skewed_lora_batch(self):
        """Highly uneven LoRA distribution, random lengths"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(BatchComposition.SKEWED, batch_size)
                )
                self.run_test_comparison(
                    x,
                    weights,
                    batch_info,
                    seq_lengths,
                    lora_assignments,
                    f"skewed batch_size={batch_size}",
                )
```
**EN:** Highly uneven LoRA distribution, random lengths This test exercises `test_highly_skewed_lora_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Highly uneven LoRA distribution, random lengths 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_highly_skewed_lora_batch`。

### Lines 771-773: supporting source context / 辅助源码上下文
```python

    # === Decode Mode Tests ===

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 774-790: test case decode uniform lora batch / 测试用例 decode uniform lora batch
```python
    def test_decode_uniform_lora_batch(self):
        """Decode mode: All sequences use same LoRA, all length 1"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(
                        BatchComposition.UNIFORM, batch_size, BatchMode.DECODE
                    )
                )
                self.run_test_comparison(
                    x,
                    weights,
                    batch_info,
                    seq_lengths,
                    lora_assignments,
                    f"decode uniform batch_size={batch_size}",
                )
```
**EN:** Decode mode: All sequences use same LoRA, all length 1 This test exercises `test_decode_uniform_lora_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Decode mode: All sequences use same LoRA, all length 1 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_uniform_lora_batch`。

### Lines 792-808: test case decode mixed lora batch / 测试用例 decode mixed lora batch
```python
    def test_decode_mixed_lora_batch(self):
        """Decode mode: Sequences distributed across LoRAs, all length 1"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(
                        BatchComposition.MIXED, batch_size, BatchMode.DECODE
                    )
                )
                self.run_test_comparison(
                    x,
                    weights,
                    batch_info,
                    seq_lengths,
                    lora_assignments,
                    f"decode mixed batch_size={batch_size}",
                )
```
**EN:** Decode mode: Sequences distributed across LoRAs, all length 1 This test exercises `test_decode_mixed_lora_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Decode mode: Sequences distributed across LoRAs, all length 1 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_mixed_lora_batch`。

### Lines 810-826: test case decode skewed lora batch / 测试用例 decode skewed lora batch
```python
    def test_decode_skewed_lora_batch(self):
        """Decode mode: Highly uneven LoRA distribution, all length 1"""
        for batch_size in [1, 2, 16, 64]:
            with self.subTest(batch_size=batch_size):
                x, weights, batch_info, seq_lengths, lora_assignments = (
                    self.create_test_batch(
                        BatchComposition.SKEWED, batch_size, BatchMode.DECODE
                    )
                )
                self.run_test_comparison(
                    x,
                    weights,
                    batch_info,
                    seq_lengths,
                    lora_assignments,
                    f"decode skewed batch_size={batch_size}",
                )
```
**EN:** Decode mode: Highly uneven LoRA distribution, all length 1 This test exercises `test_decode_skewed_lora_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Decode mode: Highly uneven LoRA distribution, all length 1 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_skewed_lora_batch`。

### Lines 829-829: class TestLmHeadPruningConsistency declaration / 类 TestLmHeadPruningConsistency 声明
```python
class TestLmHeadPruningConsistency(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 830-835: supporting statements / 辅助语句
```python
    """Verify get_lm_head_pruned_lens (LoRA) stays consistent with
    LogitsProcessor._get_pruned_states (logits_processor).

    If this test fails, it likely means one side was changed without
    updating the other. See cross-references in both functions.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 837-853: method make mock forward batch / 方法 make mock forward batch
```python
    def _make_mock_forward_batch(
        self,
        forward_mode,
        extend_seq_lens_cpu,
        return_logprob=False,
        logprob_start_lens_cpu=None,
    ):
        class MockForwardBatch:
            pass

        batch = MockForwardBatch()
        batch.forward_mode = forward_mode
        batch.batch_size = len(extend_seq_lens_cpu)
        batch.return_logprob = return_logprob
        batch.extend_seq_lens_cpu = extend_seq_lens_cpu
        batch.extend_logprob_start_lens_cpu = logprob_start_lens_cpu
        return batch
```
**EN:** This block implements `_make_mock_forward_batch` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_mock_forward_batch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 855-879: method count pruned states tokens / 方法 count pruned states tokens
```python
    def _count_pruned_states_tokens(
        self,
        forward_mode,
        extend_seq_lens_cpu,
        return_logprob=False,
        logprob_start_lens_cpu=None,
    ):
        """Call _get_pruned_states and return the number of output tokens."""
        total_tokens = sum(extend_seq_lens_cpu)
        hidden_states = torch.zeros(total_tokens, 4)

        logits_meta = LogitsMetadata(
            forward_mode=forward_mode,
            extend_return_logprob=return_logprob,
            extend_seq_lens=torch.tensor(extend_seq_lens_cpu, dtype=torch.int64),
            extend_seq_lens_cpu=extend_seq_lens_cpu,
            extend_logprob_start_lens_cpu=logprob_start_lens_cpu,
        )

        # _get_pruned_states does not use self, so pass None
        result = LogitsProcessor._get_pruned_states(
            None, hidden_states, None, None, logits_meta
        )
        pruned_states = result[0]
        return pruned_states.shape[0]
```
**EN:** Call _get_pruned_states and return the number of output tokens. This block implements `_count_pruned_states_tokens` and captures one focused piece of the module's behavior.
**CN:** Call _get_pruned_states and return the number of output tokens. 该代码块实现 `_count_pruned_states_tokens`，承担模块行为中的一个聚焦逻辑片段。

### Lines 881-914: method assert consistency / 方法 assert consistency
```python
    def _assert_consistency(
        self,
        forward_mode,
        extend_seq_lens_cpu,
        return_logprob=False,
        logprob_start_lens_cpu=None,
    ):
        mock_batch = self._make_mock_forward_batch(
            forward_mode,
            extend_seq_lens_cpu,
            return_logprob,
            logprob_start_lens_cpu,
        )
        pruned_lens = get_lm_head_pruned_lens(mock_batch)

        actual_count = self._count_pruned_states_tokens(
            forward_mode,
            extend_seq_lens_cpu,
            return_logprob,
            logprob_start_lens_cpu,
        )

        if pruned_lens is None:
            expected_count = sum(extend_seq_lens_cpu)
        else:
            expected_count = sum(pruned_lens)

        self.assertEqual(
            expected_count,
            actual_count,
            f"get_lm_head_pruned_lens expects {expected_count} tokens, "
            f"but _get_pruned_states produces {actual_count}. "
            f"These functions must stay in sync — see their cross-reference comments.",
        )
```
**EN:** This block implements `_assert_consistency` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_consistency`，承担模块行为中的一个聚焦逻辑片段。

### Lines 916-917: test case extend no logprob / 测试用例 extend no logprob
```python
    def test_extend_no_logprob(self):
        self._assert_consistency(ForwardMode.EXTEND, [4, 5, 6])
```
**EN:** This test exercises `test_extend_no_logprob` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extend_no_logprob`。

### Lines 919-925: test case extend with logprob / 测试用例 extend with logprob
```python
    def test_extend_with_logprob(self):
        self._assert_consistency(
            ForwardMode.EXTEND,
            [4, 5, 6],
            return_logprob=True,
            logprob_start_lens_cpu=[0, 5, 3],
        )
```
**EN:** This test exercises `test_extend_with_logprob` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extend_with_logprob`。

### Lines 928-929: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `reset_kernel_cache`: This block implements `reset_kernel_cache` and captures one focused piece of the module's behavior. / 该代码块实现 `reset_kernel_cache`，承担模块行为中的一个聚焦逻辑片段。
- `BatchComposition`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `BatchMode`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestChunkedSGMV`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLmHeadPruningConsistency`: Verify get_lm_head_pruned_lens (LoRA) stays consistent with LogitsProcessor._get_pruned_states (logits_processor). / 用于组织相关测试、夹具或辅助方法。
- `TestChunkedSGMV._compare_shrink_outputs`: Compare only the valid portions of shrink outputs. / 该代码块实现 `_compare_shrink_outputs`，承担模块行为中的一个聚焦逻辑片段。
- `TestChunkedSGMV.setUp`: Set up common test parameters / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestChunkedSGMV.generate_sequence_lengths`: Generate sequence lengths for a batch based on mode / 该代码块实现 `generate_sequence_lengths`，承担模块行为中的一个聚焦逻辑片段。
- `TestChunkedSGMV.create_lora_weights`: Create LoRA A and B weights for given configuration / 该辅助函数封装了 `create_lora_weights`，以便周围测试复用准备、执行或校验逻辑。
- `TestChunkedSGMV.create_batch_info`: Create LoRABatchInfo using the same logic as chunked backend / 该辅助函数封装了 `create_batch_info`，以便周围测试复用准备、执行或校验逻辑。
- `TestChunkedSGMV.stack_lora_weights`: Stack LoRA weights from different adapters into a single tensor / 该代码块实现 `stack_lora_weights`，承担模块行为中的一个聚焦逻辑片段。
- `TestChunkedSGMV.create_embedding_lora_a_weights`: Create LoRA A weights for embedding lookup. / 该辅助函数封装了 `create_embedding_lora_a_weights`，以便周围测试复用准备、执行或校验逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `unittest`, `enum`, `typing`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers.logits_processor`, `sglang.srt.lora.backend.chunked_backend`, `sglang.srt.lora.triton_ops`, `sglang.srt.lora.triton_ops.chunked_sgmv_expand`, `sglang.srt.lora.triton_ops.chunked_sgmv_shrink`, `sglang.srt.lora.utils`, `sglang.srt.model_executor.forward_batch_info`, `sglang.test.ci.ci_register`, `sglang.test.lora_utils`

- **Total lines / 总行数**: 929
