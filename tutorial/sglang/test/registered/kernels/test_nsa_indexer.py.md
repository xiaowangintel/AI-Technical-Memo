# test_nsa_indexer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/kernels/test_nsa_indexer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates nsa indexer behavior in SGLang's kernels area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 kernels 领域中与 nsa indexer 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
import unittest
from typing import List, Optional, Tuple
from unittest.mock import MagicMock, patch

import torch

from sglang.srt.layers import dp_attention as _dp_attn
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `typing`, `unittest.mock`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `typing`, `unittest.mock`, `torch`。

### Lines 9-10: supporting source context / 辅助源码上下文
```python

# Patch DP-attention globals before importing backends
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 11-11: supporting statements / 辅助语句
```python
_dp_attn.get_attention_tp_size = lambda: 1  # TP size = 1 for unit test
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 13-25: module imports and dependencies / 模块导入与依赖
```python
from sglang.srt.configs.model_config import AttentionArch
from sglang.srt.layers.attention.nsa.nsa_indexer import (
    BaseIndexerMetadata,
    Indexer,
    rotate_activation,
)
from sglang.srt.layers.attention.nsa_backend import NativeSparseAttnBackend
from sglang.srt.layers.layernorm import LayerNorm
from sglang.srt.layers.linear import LinearBase
from sglang.srt.mem_cache.memory_pool import NSATokenToKVPool
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.srt.configs.model_config`, `sglang.srt.layers.attention.nsa.nsa_indexer`, `sglang.srt.layers.attention.nsa_backend`, `sglang.srt.layers.layernorm`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.srt.configs.model_config`, `sglang.srt.layers.attention.nsa.nsa_indexer`, `sglang.srt.layers.attention.nsa_backend`, `sglang.srt.layers.layernorm`。

### Lines 27-49: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=18, stage="base-b", runner_config="1-gpu-large")

# Global configuration for all indexer tests
DEFAULT_CONFIG = {
    "device": "cuda",
    "dtype": torch.bfloat16,
    "kv_cache_dtype": torch.float8_e4m3fn,
    "context_len": 2048,
    "max_bs": 64,
    "hidden_size": 5120,
    "index_n_heads": 32,
    "index_head_dim": 128,
    "rope_head_dim": 64,
    "index_topk": 64,
    "q_lora_rank": 1536,
    "kv_lora_rank": 512,
    "qk_rope_head_dim": 64,
    "qk_nope_head_dim": 128,
    "max_position_embeddings": 163840,
    "rope_theta": 10000.0,
    "layer_id": 0,
    "page_size": 64,
}
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 52-52: class MockIndexerMetadata declaration / 类 MockIndexerMetadata 声明
```python
class MockIndexerMetadata(BaseIndexerMetadata):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `BaseIndexerMetadata`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `BaseIndexerMetadata`。

### Lines 53-53: supporting statements / 辅助语句
```python
    """Mock implementation of BaseIndexerMetadata for testing."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 55-59: method init / 方法 init
```python
    def __init__(self, batch_size, seq_lens, page_table=None):
        self.batch_size = batch_size
        self.seq_lens = seq_lens
        self.page_table = page_table
        self.device = "cuda"
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 61-63: helper routine get seqlens int32 / 辅助流程 get seqlens int32
```python
    def get_seqlens_int32(self) -> torch.Tensor:
        """Return: (batch_size,) int32 tensor"""
        return torch.tensor(self.seq_lens, dtype=torch.int32, device=self.device)
```
**EN:** Return: (batch_size,) int32 tensor This helper encapsulates `get_seqlens_int32` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: (batch_size,) int32 tensor 该辅助函数封装了 `get_seqlens_int32`，以便周围测试复用准备、执行或校验逻辑。

### Lines 65-81: helper routine get page table 64 / 辅助流程 get page table 64
```python
    def get_page_table_64(self) -> torch.Tensor:
        """Return: (batch_size, num_blocks) int32, page table with page size 64."""
        if self.page_table is not None:
            return self.page_table
        # Create a simple page table for testing
        max_seq_len = max(self.seq_lens)
        num_blocks = (max_seq_len + 63) // 64  # Round up to page size 64
        page_table = torch.zeros(
            (self.batch_size, num_blocks), dtype=torch.int32, device=self.device
        )
        for i in range(self.batch_size):
            # Simple linear mapping: block i maps to page i
            num_blocks_needed = (self.seq_lens[i] + 63) // 64
            page_table[i, :num_blocks_needed] = torch.arange(
                num_blocks_needed, device=self.device
            )
        return page_table
```
**EN:** Return: (batch_size, num_blocks) int32, page table with page size 64. This helper encapsulates `get_page_table_64` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: (batch_size, num_blocks) int32, page table with page size 64. 该辅助函数封装了 `get_page_table_64`，以便周围测试复用准备、执行或校验逻辑。

### Lines 83-97: helper routine get page table 1 / 辅助流程 get page table 1
```python
    def get_page_table_1(self) -> torch.Tensor:
        """Return: (batch_size, num_blocks) int32, page table with page size 1."""
        # Create a simple page table for testing with page size 1
        max_seq_len = max(self.seq_lens)
        num_blocks = max_seq_len  # Page size 1 means num_blocks == max_seq_len
        page_table = torch.zeros(
            (self.batch_size, num_blocks), dtype=torch.int32, device=self.device
        )
        for i in range(self.batch_size):
            # Simple linear mapping: block i maps to page i
            num_blocks_needed = self.seq_lens[i]
            page_table[i, :num_blocks_needed] = torch.arange(
                num_blocks_needed, device=self.device
            )
        return page_table
```
**EN:** Return: (batch_size, num_blocks) int32, page table with page size 1. This helper encapsulates `get_page_table_1` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: (batch_size, num_blocks) int32, page table with page size 1. 该辅助函数封装了 `get_page_table_1`，以便周围测试复用准备、执行或校验逻辑。

### Lines 99-106: helper routine get seqlens expanded / 辅助流程 get seqlens expanded
```python
    def get_seqlens_expanded(self) -> torch.Tensor:
        """Return: (sum_extend_seq_len,) int32 tensor"""
        # For extend mode, each new token attends to progressively more tokens
        # For a sequence being extended from position 0 to seq_len, token i attends to i+1 tokens
        result = []
        for seq_len in self.seq_lens:
            result.extend(range(1, seq_len + 1))
        return torch.tensor(result, dtype=torch.int32, device=self.device)
```
**EN:** Return: (sum_extend_seq_len,) int32 tensor This helper encapsulates `get_seqlens_expanded` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: (sum_extend_seq_len,) int32 tensor 该辅助函数封装了 `get_seqlens_expanded`，以便周围测试复用准备、执行或校验逻辑。

### Lines 108-129: helper routine get indexer kvcache range / 辅助流程 get indexer kvcache range
```python
    def get_indexer_kvcache_range(self) -> Tuple[torch.Tensor, torch.Tensor]:
        """
        Return: (tokens, ), (tokens, ) int32, k_start and k_end in kv cache for each token.
        For extend mode, token i attends to tokens [0, i].
        """
        ks_list = []
        ke_list = []
        k_offset = 0
        for seq_len in self.seq_lens:
            # For a sequence being extended from position 0 to seq_len
            # Token i attends to [k_offset, k_offset + i + 1)
            ks = torch.full((seq_len,), k_offset, dtype=torch.int32, device=self.device)
            ke = torch.arange(
                k_offset + 1,
                k_offset + seq_len + 1,
                dtype=torch.int32,
                device=self.device,
            )
            ks_list.append(ks)
            ke_list.append(ke)
            k_offset += seq_len
        return torch.cat(ks_list, dim=0), torch.cat(ke_list, dim=0)
```
**EN:** Return: (tokens, ), (tokens, ) int32, k_start and k_end in kv cache for each token. This helper encapsulates `get_indexer_kvcache_range` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: (tokens, ), (tokens, ) int32, k_start and k_end in kv cache for each token. 该辅助函数封装了 `get_indexer_kvcache_range`，以便周围测试复用准备、执行或校验逻辑。

### Lines 131-133: helper routine get indexer seq len cpu / 辅助流程 get indexer seq len cpu
```python
    def get_indexer_seq_len_cpu(self) -> torch.Tensor:
        """Return: seq lens for each batch."""
        return torch.tensor(self.seq_lens, dtype=torch.int32, device="cpu")
```
**EN:** Return: seq lens for each batch. This helper encapsulates `get_indexer_seq_len_cpu` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: seq lens for each batch. 该辅助函数封装了 `get_indexer_seq_len_cpu`，以便周围测试复用准备、执行或校验逻辑。

### Lines 135-137: helper routine get indexer seq len / 辅助流程 get indexer seq len
```python
    def get_indexer_seq_len(self) -> torch.Tensor:
        """Return: seq lens for each batch."""
        return torch.tensor(self.seq_lens, dtype=torch.int32, device=self.device)
```
**EN:** Return: seq lens for each batch. This helper encapsulates `get_indexer_seq_len` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: seq lens for each batch. 该辅助函数封装了 `get_indexer_seq_len`，以便周围测试复用准备、执行或校验逻辑。

### Lines 139-143: helper routine get nsa extend len cpu / 辅助流程 get nsa extend len cpu
```python
    def get_nsa_extend_len_cpu(self) -> List[int]:
        """
        Return: extend seq lens for each batch.
        """
        return list(self.seq_lens)
```
**EN:** Return: extend seq lens for each batch. This helper encapsulates `get_nsa_extend_len_cpu` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: extend seq lens for each batch. 该辅助函数封装了 `get_nsa_extend_len_cpu`，以便周围测试复用准备、执行或校验逻辑。

### Lines 145-150: helper routine get token to batch idx / 辅助流程 get token to batch idx
```python
    def get_token_to_batch_idx(self) -> torch.Tensor:
        """Return: batch idx for each token."""
        result = []
        for batch_idx, seq_len in enumerate(self.seq_lens):
            result.extend([batch_idx] * seq_len)
        return torch.tensor(result, dtype=torch.int32, device=self.device)
```
**EN:** Return: batch idx for each token. This helper encapsulates `get_token_to_batch_idx` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Return: batch idx for each token. 该辅助函数封装了 `get_token_to_batch_idx`，以便周围测试复用准备、执行或校验逻辑。

### Lines 152-166: method topk transform / 方法 topk transform
```python
    def topk_transform(
        self,
        logits: torch.Tensor,
        topk: int,
        ks: Optional[torch.Tensor] = None,
        cu_seqlens_q: Optional[torch.Tensor] = None,
        ke_offset: Optional[torch.Tensor] = None,
        batch_idx_list: Optional[torch.Tensor] = None,
        topk_indices_offset_override: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        """
        Perform topk selection on the logits.
        For testing, just return the topk indices.
        """
        return torch.topk(logits, k=topk, dim=-1).indices
```
**EN:** Perform topk selection on the logits. This block implements `topk_transform` and captures one focused piece of the module's behavior.
**CN:** Perform topk selection on the logits. 该代码块实现 `topk_transform`，承担模块行为中的一个聚焦逻辑片段。

### Lines 169-169: class MockModelRunner declaration / 类 MockModelRunner 声明
```python
class MockModelRunner:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 170-249: method init (part 1/2) / 方法 init（第 1/2 部分）
```python
    def __init__(self, config=None):
        self.device = "cuda"
        self.config = {**DEFAULT_CONFIG, **(config or {})}
        self.dtype = self.config["dtype"]
        self.kv_cache_dtype = self.config["kv_cache_dtype"]
        self.is_hybrid_swa = False

        # Model configuration
        attention_arch = AttentionArch.MLA
        max_context_len = self.config["context_len"]
        max_batch_size = self.config["max_bs"]

        # Create mock hf_config for NSA - instantiate it as an object, not a type
        hf_config = type(
            "HfConfig",
            (),
            {
                "architectures": ["DeepseekV3ForCausalLM"],
                "index_topk": self.config["index_topk"],
                "index_head_dim": self.config["index_head_dim"],
                "index_n_heads": self.config["index_n_heads"],
            },
        )()

        self.model_config = type(
            "ModelConfig",
            (),
            {
                "context_len": max_context_len,
                "is_multimodal": False,
                "attention_arch": attention_arch,
                "num_attention_heads": 128,
                "kv_lora_rank": self.config["kv_lora_rank"],
                "qk_rope_head_dim": self.config["qk_rope_head_dim"],
                "qk_nope_head_dim": self.config["qk_nope_head_dim"],
                "hf_config": hf_config,
            },
        )()

        self.sliding_window_size = None
        self.page_size = self.config["page_size"]

        # Create req_to_token_pool
        self.req_to_token_pool = type(
            "TokenPool",
            (),
            {
                "size": max_batch_size,
                "req_to_token": torch.zeros(
                    max_batch_size,
                    max_context_len,
                    dtype=torch.int32,
                    device=self.device,
                ),
            },
        )()

        # Create NSATokenToKVPool
        max_total_num_tokens = max_batch_size * max_context_len
        self.token_to_kv_pool = NSATokenToKVPool(
            size=max_total_num_tokens,
            page_size=self.config["page_size"],
            dtype=self.config["kv_cache_dtype"],
            kv_lora_rank=self.config["kv_lora_rank"],
            qk_rope_head_dim=self.config["qk_rope_head_dim"],
            layer_num=1,
            device=self.device,
            index_head_dim=self.config["index_head_dim"],
            enable_memory_saver=False,
            kv_cache_dim=self.config["kv_lora_rank"] + self.config["qk_rope_head_dim"],
        )

        # Required by backend with NSA-specific attributes
        self.server_args = type(
            "ServerArgs",
            (),
            {
                "kv_cache_dtype": "auto",
                "speculative_eagle_topk": None,
                "speculative_num_draft_tokens": 0,
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 250-254: method init (part 2/2) / 方法 init（第 2/2 部分）
```python
                "enable_deterministic_inference": False,
                "nsa_prefill_backend": "flashmla_sparse",
                "nsa_decode_backend": "fa3",
            },
        )()
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 257-258: class TestNSAIndexer declaration / 类 TestNSAIndexer 声明
```python
@unittest.skipIf(not torch.cuda.is_available(), "Test requires CUDA")
class TestNSAIndexer(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 259-271: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        """Set up global server args for testing."""
        server_args = ServerArgs(model_path="dummy")
        server_args.enable_dp_attention = False
        server_args.nsa_prefill_backend = "flashmla_sparse"
        server_args.nsa_decode_backend = "flashmla_sparse"
        set_global_server_args_for_scheduler(server_args)

        # Check GPU capability for FP8
        if torch.cuda.is_available():
            compute_capability = torch.cuda.get_device_capability()
            cls.supports_fp8 = compute_capability[0] >= 9  # Hopper or newer
```
**EN:** Set up global server args for testing. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up global server args for testing. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 273-276: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        """Clean up after all tests."""
        pass
```
**EN:** Clean up after all tests. This routine releases resources and restores state after the related tests finish.
**CN:** Clean up after all tests. 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 278-284: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        # Test parameters
        self.batch_size = 2
        self.seq_len = 128
        self.config = DEFAULT_CONFIG.copy()
        self.device = "cuda"
        self.dtype = torch.bfloat16
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 286-292: method init model runner / 方法 init model runner
```python
    def _init_model_runner(self, config_override=None):
        """Initialize model runner with optional config override."""
        config = self.config.copy()
        if config_override:
            config.update(config_override)
        self.model_runner = MockModelRunner(config)
        self.backend = NativeSparseAttnBackend(self.model_runner)
```
**EN:** Initialize model runner with optional config override. This block implements `_init_model_runner` and captures one focused piece of the module's behavior.
**CN:** Initialize model runner with optional config override. 该代码块实现 `_init_model_runner`，承担模块行为中的一个聚焦逻辑片段。

### Lines 294-327: method create indexer / 方法 create indexer
```python
    def _create_indexer(self, **kwargs):
        """Create an Indexer instance with default parameters."""
        params = {
            "hidden_size": self.config["hidden_size"],
            "index_n_heads": self.config["index_n_heads"],
            "index_head_dim": self.config["index_head_dim"],
            "rope_head_dim": self.config["rope_head_dim"],
            "index_topk": self.config["index_topk"],
            "q_lora_rank": self.config["q_lora_rank"],
            "max_position_embeddings": self.config["max_position_embeddings"],
            "rope_theta": self.config["rope_theta"],
            "layer_id": self.config["layer_id"],
            "scale_fmt": "ue8m0",
            "block_size": 128,
            "quant_config": None,  # No quantization for testing
        }
        params.update(kwargs)

        torch.set_default_dtype(self.dtype)
        indexer = Indexer(**params)
        # Move indexer to CUDA device
        indexer = indexer.to(device=self.device)

        # Convert linear layer weights to bfloat16 (but preserve LayerNorm's float32
        # and weights_proj's float32 - it uses params_dtype=torch.float32 in production)
        # Need to recursively convert LinearBase submodules (like ReplicatedLinear)
        for name, module in indexer.named_modules():
            # Check for LinearBase (parent of ReplicatedLinear) but exclude LayerNorm
            # Also exclude weights_proj which uses float32 params in production
            if isinstance(module, LinearBase) and not isinstance(module, LayerNorm):
                if "weights_proj" not in name:
                    module.to(dtype=self.dtype)

        return indexer
```
**EN:** Create an Indexer instance with default parameters. This block implements `_create_indexer` and captures one focused piece of the module's behavior.
**CN:** Create an Indexer instance with default parameters. 该代码块实现 `_create_indexer`，承担模块行为中的一个聚焦逻辑片段。

### Lines 329-398: method create forward batch / 方法 create forward batch
```python
    def _create_forward_batch(
        self, mode, batch_size=None, seq_len=None, extend_len=None
    ):
        """Create a forward batch for testing."""
        batch_size = batch_size or self.batch_size
        seq_len = seq_len or self.seq_len

        if mode == ForwardMode.EXTEND:
            q_len = extend_len or seq_len
            total_len = seq_len

            forward_batch = ForwardBatch(
                batch_size=batch_size,
                input_ids=torch.randint(
                    0, 100, (batch_size, q_len), device=self.device
                ),
                out_cache_loc=torch.arange(
                    batch_size * (total_len - q_len),
                    batch_size * total_len,
                    device=self.device,
                ),
                seq_lens_sum=batch_size * total_len,
                forward_mode=mode,
                req_pool_indices=torch.arange(batch_size, device=self.device),
                seq_lens=torch.tensor([total_len] * batch_size, device=self.device),
                seq_lens_cpu=torch.tensor([total_len] * batch_size, device="cpu"),
                extend_prefix_lens=torch.tensor(
                    [total_len - q_len] * batch_size, device=self.device
                ),
                extend_prefix_lens_cpu=torch.tensor(
                    [total_len - q_len] * batch_size, device="cpu"
                ),
                extend_seq_lens=torch.tensor([q_len] * batch_size, device=self.device),
                extend_seq_lens_cpu=torch.tensor([q_len] * batch_size, device="cpu"),
                attn_backend=self.backend,
            )
        else:  # ForwardMode.DECODE
            decode_len = 1
            total_len = seq_len + decode_len

            forward_batch = ForwardBatch(
                batch_size=batch_size,
                input_ids=torch.randint(
                    0, 100, (batch_size, decode_len), device=self.device
                ),
                out_cache_loc=torch.arange(
                    batch_size * seq_len, batch_size * total_len, device=self.device
                ),
                seq_lens_sum=batch_size * total_len,
                forward_mode=mode,
                req_pool_indices=torch.arange(batch_size, device=self.device),
                seq_lens=torch.tensor([total_len] * batch_size, device=self.device),
                seq_lens_cpu=torch.tensor([total_len] * batch_size, device="cpu"),
                attn_backend=self.backend,
            )

        # Add token pools
        forward_batch.req_to_token_pool = self.model_runner.req_to_token_pool
        forward_batch.token_to_kv_pool = self.model_runner.token_to_kv_pool

        # Mock write to req_to_token_pool
        page_size = self.model_runner.page_size
        for i in range(batch_size):
            seq_length = total_len
            for j in range(seq_length):
                self.model_runner.req_to_token_pool.req_to_token[i, j] = (
                    i * seq_length + j + page_size
                )

        return forward_batch
```
**EN:** Create a forward batch for testing. This block implements `_create_forward_batch` and captures one focused piece of the module's behavior.
**CN:** Create a forward batch for testing. 该代码块实现 `_create_forward_batch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 400-418: method verify topk output / 方法 verify topk output
```python
    def _verify_topk_output(self, topk_indices, batch_size, q_len, topk):
        """Verify the topk indices output shape and basic properties."""
        self.assertIsNotNone(topk_indices)
        self.assertEqual(topk_indices.device.type, "cuda")

        # Check shape - should be (total_q_len, topk_padded)
        # where topk_padded is aligned to 2048
        self.assertEqual(len(topk_indices.shape), 2)
        self.assertEqual(topk_indices.shape[0], batch_size * q_len)

        # Check that topk is padded to at least topk
        self.assertGreaterEqual(topk_indices.shape[1], topk)

        # Check for padding values (-1)
        has_padding = (topk_indices == -1).any()
        self.assertTrue(
            has_padding or topk_indices.shape[1] == topk,
            "Output should have padding or exact topk size",
        )
```
**EN:** Verify the topk indices output shape and basic properties. This block implements `_verify_topk_output` and captures one focused piece of the module's behavior.
**CN:** Verify the topk indices output shape and basic properties. 该代码块实现 `_verify_topk_output`，承担模块行为中的一个聚焦逻辑片段。

### Lines 420-432: test case indexer basic creation / 测试用例 indexer basic creation
```python
    @patch("sglang.srt.layers.attention.nsa.nsa_indexer.deep_gemm")
    def test_indexer_basic_creation(self, mock_deep_gemm):
        """Test basic indexer creation and initialization."""
        mock_deep_gemm.get_num_sms.return_value = 132

        indexer = self._create_indexer()

        self.assertEqual(indexer.hidden_size, self.config["hidden_size"])
        self.assertEqual(indexer.n_heads, self.config["index_n_heads"])
        self.assertEqual(indexer.head_dim, self.config["index_head_dim"])
        self.assertEqual(indexer.rope_head_dim, self.config["rope_head_dim"])
        self.assertEqual(indexer.index_topk, self.config["index_topk"])
        self.assertEqual(indexer.layer_id, self.config["layer_id"])
```
**EN:** Test basic indexer creation and initialization. This test exercises `test_indexer_basic_creation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic indexer creation and initialization. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_indexer_basic_creation`。

### Lines 434-513: test case forward extend mode (part 1/2) / 测试用例 forward extend mode（第 1/2 部分）
```python
    @patch("sglang.srt.layers.attention.nsa.nsa_indexer.deep_gemm")
    @patch("sglang.srt.layers.attention.nsa.triton_kernel.act_quant")
    def test_forward_extend_mode(self, mock_act_quant, mock_deep_gemm):
        """Test indexer forward pass in extend mode."""
        if not self.supports_fp8:
            self.skipTest("FP8 requires Hopper GPU or newer")

        # Setup mocks
        mock_deep_gemm.get_num_sms.return_value = 132
        mock_deep_gemm.get_paged_mqa_logits_metadata.return_value = MagicMock()

        def mock_quant(x, *args, **kwargs):
            # Return FP8 tensor and scale
            return x.to(torch.float8_e4m3fn), torch.ones(
                x.shape[0], dtype=torch.float32, device=x.device
            )

        mock_act_quant.side_effect = mock_quant

        # Mock deep_gemm.fp8_mqa_logits to return logits (ragged path)
        def mock_mqa_logits(q, kv, weights, ks, ke, *args, **kwargs):
            # q shape: (sum_extend_seq_len, ...), return logits for each query token
            num_queries = q.shape[0]
            # kv is a tuple (k_fp8, k_scale), get total number of keys from k_fp8
            k_fp8, k_scale = kv
            max_kv_len = k_fp8.shape[0]  # Total keys across all batches (k_offset)
            return torch.randn(
                num_queries, max_kv_len, dtype=torch.float32, device="cuda"
            )

        mock_deep_gemm.fp8_mqa_logits.side_effect = mock_mqa_logits

        # Also mock the paged version for completeness
        def mock_paged_mqa_logits(q, kv, weights, *args, **kwargs):
            batch_size = q.shape[0]
            seq_len = 128
            return torch.randn(batch_size, seq_len, dtype=torch.float32, device="cuda")

        mock_deep_gemm.fp8_paged_mqa_logits.side_effect = mock_paged_mqa_logits

        self._init_model_runner()

        indexer = self._create_indexer()
        forward_batch = self._create_forward_batch(ForwardMode.EXTEND)

        # Create input tensors
        total_tokens = self.batch_size * self.seq_len
        hidden_states = torch.randn(
            total_tokens,
            self.config["hidden_size"],
            dtype=self.dtype,
            device=self.device,
        )
        q_lora = torch.randn(
            total_tokens,
            self.config["q_lora_rank"],
            dtype=self.dtype,
            device=self.device,
        )
        positions = torch.arange(total_tokens, device=self.device)

        # Run forward pass
        with patch.object(
            self.backend,
            "get_indexer_metadata",
            return_value=MockIndexerMetadata(
                self.batch_size, [self.seq_len] * self.batch_size
            ),
        ):
            topk_indices = indexer(
                x=hidden_states,
                q_lora=q_lora,
                positions=positions,
                forward_batch=forward_batch,
                layer_id=self.config["layer_id"],
            )

        # Verify output
        self._verify_topk_output(
            topk_indices, self.batch_size, self.seq_len, self.config["index_topk"]
```
**EN:** Test indexer forward pass in extend mode. This test exercises `test_forward_extend_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test indexer forward pass in extend mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_forward_extend_mode`。 这一段对应同一逻辑块的第 1 部分。

### Lines 514-514: test case forward extend mode (part 2/2) / 测试用例 forward extend mode（第 2/2 部分）
```python
        )
```
**EN:** Test indexer forward pass in extend mode. This test exercises `test_forward_extend_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test indexer forward pass in extend mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_forward_extend_mode`。 这一段对应同一逻辑块的第 2 部分。

### Lines 516-580: test case forward decode mode / 测试用例 forward decode mode
```python
    @patch("sglang.srt.layers.attention.nsa.nsa_indexer.deep_gemm")
    @patch("sglang.srt.layers.attention.nsa.triton_kernel.act_quant")
    def test_forward_decode_mode(self, mock_act_quant, mock_deep_gemm):
        """Test indexer forward pass in decode mode."""
        if not self.supports_fp8:
            self.skipTest("FP8 requires Hopper GPU or newer")

        # Setup mocks
        mock_deep_gemm.get_num_sms.return_value = 132
        mock_deep_gemm.get_paged_mqa_logits_metadata.return_value = MagicMock()

        def mock_quant(x, *args, **kwargs):
            return x.to(torch.float8_e4m3fn), torch.ones(
                x.shape[0], dtype=torch.float32, device=x.device
            )

        mock_act_quant.side_effect = mock_quant

        def mock_paged_mqa_logits(q, kv, weights, *args, **kwargs):
            batch_size = q.shape[0]
            seq_len = 128
            return torch.randn(batch_size, seq_len, dtype=torch.float32, device="cuda")

        mock_deep_gemm.fp8_paged_mqa_logits.side_effect = mock_paged_mqa_logits

        self._init_model_runner()

        indexer = self._create_indexer()
        forward_batch = self._create_forward_batch(ForwardMode.DECODE)

        # Create input tensors for decode (batch_size tokens only)
        hidden_states = torch.randn(
            self.batch_size,
            self.config["hidden_size"],
            dtype=self.dtype,
            device=self.device,
        )
        q_lora = torch.randn(
            self.batch_size,
            self.config["q_lora_rank"],
            dtype=self.dtype,
            device=self.device,
        )
        positions = torch.arange(self.batch_size, device=self.device)

        # Run forward pass
        with patch.object(
            self.backend,
            "get_indexer_metadata",
            return_value=MockIndexerMetadata(
                self.batch_size, [self.seq_len + 1] * self.batch_size
            ),
        ):
            topk_indices = indexer(
                x=hidden_states,
                q_lora=q_lora,
                positions=positions,
                forward_batch=forward_batch,
                layer_id=self.config["layer_id"],
            )

        # Verify output - decode mode has q_len=1
        self._verify_topk_output(
            topk_indices, self.batch_size, 1, self.config["index_topk"]
        )
```
**EN:** Test indexer forward pass in decode mode. This test exercises `test_forward_decode_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test indexer forward pass in decode mode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_forward_decode_mode`。

### Lines 582-593: test case rotate activation / 测试用例 rotate activation
```python
    def test_rotate_activation(self):
        """Test the Hadamard transform (rotate_activation) function."""
        # Test with power-of-2 hidden size
        hidden_size = 128
        x = torch.randn(16, hidden_size, dtype=torch.bfloat16, device=self.device)

        try:
            output = rotate_activation(x)
            self.assertEqual(output.shape, x.shape)
            self.assertEqual(output.dtype, torch.bfloat16)
        except Exception:
            self.skipTest("hadamard JIT kernel not available")
```
**EN:** Test the Hadamard transform (rotate_activation) function. This test exercises `test_rotate_activation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the Hadamard transform (rotate_activation) function. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rotate_activation`。

### Lines 595-602: test case rotate activation invalid size / 测试用例 rotate activation invalid size
```python
    def test_rotate_activation_invalid_size(self):
        """Test that rotate_activation fails with non-power-of-2 size."""
        # Test with non-power-of-2 hidden size
        hidden_size = 129  # Not a power of 2
        x = torch.randn(16, hidden_size, dtype=torch.bfloat16, device=self.device)

        with self.assertRaises(AssertionError):
            rotate_activation(x)
```
**EN:** Test that rotate_activation fails with non-power-of-2 size. This test exercises `test_rotate_activation_invalid_size` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that rotate_activation fails with non-power-of-2 size. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rotate_activation_invalid_size`。

### Lines 604-627: test case indexer metadata interface / 测试用例 indexer metadata interface
```python
    def test_indexer_metadata_interface(self):
        """Test the BaseIndexerMetadata interface implementation."""
        batch_size = 4
        seq_lens = [64, 128, 96, 112]

        metadata = MockIndexerMetadata(batch_size, seq_lens)

        # Test get_seqlens_int32
        seqlens = metadata.get_seqlens_int32()
        self.assertEqual(seqlens.shape, (batch_size,))
        self.assertEqual(seqlens.dtype, torch.int32)
        self.assertTrue(torch.all(seqlens == torch.tensor(seq_lens, device="cuda")))

        # Test get_page_table_64
        page_table = metadata.get_page_table_64()
        self.assertEqual(len(page_table.shape), 2)
        self.assertEqual(page_table.shape[0], batch_size)
        self.assertEqual(page_table.dtype, torch.int32)

        # Test topk_transform
        logits = torch.randn(batch_size, 128, device="cuda")
        topk = 64
        topk_indices = metadata.topk_transform(logits, topk)
        self.assertEqual(topk_indices.shape, (batch_size, topk))
```
**EN:** Test the BaseIndexerMetadata interface implementation. This test exercises `test_indexer_metadata_interface` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the BaseIndexerMetadata interface implementation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_indexer_metadata_interface`。

### Lines 628-639: supporting source context / 辅助源码上下文
```python

    # TODO: enable this test after indexer accuracy aligned
    # @patch("sglang.srt.layers.attention.nsa.nsa_indexer.deep_gemm")
    # def test_indexer_with_different_topk(self, mock_deep_gemm):
    #     """Test indexer with different topk values."""
    #     mock_deep_gemm.get_num_sms.return_value = 132

    #     for topk in [32, 64, 128]:
    #         with self.subTest(topk=topk):
    #             indexer = self._create_indexer(index_topk=topk)
    #             self.assertEqual(indexer.index_topk, topk)

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 640-648: test case indexer with fused wk / 测试用例 indexer with fused wk
```python
    @patch("sglang.srt.layers.attention.nsa.nsa_indexer.deep_gemm")
    def test_indexer_with_fused_wk(self, mock_deep_gemm):
        """Test indexer creation with fused wk and weights projection."""
        mock_deep_gemm.get_num_sms.return_value = 132

        # Note: fuse_wk_and_weights_proj feature is not currently implemented
        # This test verifies basic indexer creation still works
        indexer = self._create_indexer()
        self.assertIsNotNone(indexer)
```
**EN:** Test indexer creation with fused wk and weights projection. This test exercises `test_indexer_with_fused_wk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test indexer creation with fused wk and weights projection. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_indexer_with_fused_wk`。

### Lines 650-657: test case indexer with alt stream / 测试用例 indexer with alt stream
```python
    @patch("sglang.srt.layers.attention.nsa.nsa_indexer.deep_gemm")
    def test_indexer_with_alt_stream(self, mock_deep_gemm):
        """Test indexer creation with alternative CUDA stream."""
        mock_deep_gemm.get_num_sms.return_value = 132

        alt_stream = torch.cuda.Stream()
        indexer = self._create_indexer(alt_stream=alt_stream)
        self.assertEqual(indexer.alt_stream, alt_stream)
```
**EN:** Test indexer creation with alternative CUDA stream. This test exercises `test_indexer_with_alt_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test indexer creation with alternative CUDA stream. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_indexer_with_alt_stream`。

### Lines 659-681: test case shape sanity checks / 测试用例 shape sanity checks
```python
    def test_shape_sanity_checks(self):
        """Test various shape combinations for consistency."""
        test_configs = [
            {"batch_size": 1, "seq_len": 64},
            {"batch_size": 4, "seq_len": 128},
            {"batch_size": 8, "seq_len": 256},
        ]

        for config in test_configs:
            with self.subTest(**config):
                batch_size = config["batch_size"]
                seq_len = config["seq_len"]

                # Test metadata shapes
                metadata = MockIndexerMetadata(batch_size, [seq_len] * batch_size)

                seqlens = metadata.get_seqlens_int32()
                self.assertEqual(seqlens.shape, (batch_size,))

                page_table = metadata.get_page_table_64()
                expected_blocks = (seq_len + 63) // 64
                self.assertEqual(page_table.shape[0], batch_size)
                self.assertGreaterEqual(page_table.shape[1], expected_blocks)
```
**EN:** Test various shape combinations for consistency. This test exercises `test_shape_sanity_checks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test various shape combinations for consistency. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_shape_sanity_checks`。

### Lines 684-685: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `MockIndexerMetadata`: Mock implementation of BaseIndexerMetadata for testing. / 用于组织相关测试、夹具或辅助方法。
- `MockModelRunner`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNSAIndexer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `MockIndexerMetadata.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `MockIndexerMetadata.get_seqlens_int32`: Return: (batch_size,) int32 tensor / 该辅助函数封装了 `get_seqlens_int32`，以便周围测试复用准备、执行或校验逻辑。
- `MockIndexerMetadata.get_page_table_64`: Return: (batch_size, num_blocks) int32, page table with page size 64. / 该辅助函数封装了 `get_page_table_64`，以便周围测试复用准备、执行或校验逻辑。
- `MockIndexerMetadata.get_page_table_1`: Return: (batch_size, num_blocks) int32, page table with page size 1. / 该辅助函数封装了 `get_page_table_1`，以便周围测试复用准备、执行或校验逻辑。
- `MockIndexerMetadata.get_seqlens_expanded`: Return: (sum_extend_seq_len,) int32 tensor / 该辅助函数封装了 `get_seqlens_expanded`，以便周围测试复用准备、执行或校验逻辑。
- `MockIndexerMetadata.get_indexer_kvcache_range`: Return: (tokens, ), (tokens, ) int32, k_start and k_end in kv cache for each token. / 该辅助函数封装了 `get_indexer_kvcache_range`，以便周围测试复用准备、执行或校验逻辑。
- `MockIndexerMetadata.get_indexer_seq_len_cpu`: Return: seq lens for each batch. / 该辅助函数封装了 `get_indexer_seq_len_cpu`，以便周围测试复用准备、执行或校验逻辑。
- `MockIndexerMetadata.get_indexer_seq_len`: Return: seq lens for each batch. / 该辅助函数封装了 `get_indexer_seq_len`，以便周围测试复用准备、执行或校验逻辑。
- `MockIndexerMetadata.get_nsa_extend_len_cpu`: Return: extend seq lens for each batch. / 该辅助函数封装了 `get_nsa_extend_len_cpu`，以便周围测试复用准备、执行或校验逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `typing`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers`, `sglang.test.ci.ci_register`, `sglang.srt.configs.model_config`, `sglang.srt.layers.attention.nsa.nsa_indexer`, `sglang.srt.layers.attention.nsa_backend`, `sglang.srt.layers.layernorm`, `sglang.srt.layers.linear`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.server_args`, `sglang.test.test_utils`

- **Total lines / 总行数**: 685
