# test_trtllm_mla_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/attention/test_trtllm_mla_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `trtllm mla backend` scenario in `test/manual/attention`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/attention` 中的 `trtllm mla backend` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-67: Constants and scenario settings / 常量与场景配置
```python
import math
import unittest

import numpy as np
import torch

from sglang.srt.layers import dp_attention as _dp_attn

# Patch DP-attention globals before importing backends
# TODO: change the interface of both trtllm_mla and flashinfer backends to take tp_size as an argument instead of patching
_dp_attn.get_attention_tp_size = lambda: 1  # TP size = 1 for unit test

from sglang.srt.configs.model_config import AttentionArch
from sglang.srt.layers.attention.flashinfer_mla_backend import FlashInferMLAAttnBackend
from sglang.srt.layers.attention.trtllm_mla_backend import (
    TRTLLMMLABackend,
    TRTLLMMLADecodeMetadata,
)
from sglang.srt.layers.attention.utils import get_num_page_per_block_flashmla
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.mem_cache.memory_pool import MLATokenToKVPool
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.server_args import (
    ServerArgs,
    get_global_server_args,
    set_global_server_args_for_scheduler,
)
from sglang.srt.utils import is_flashinfer_available
from sglang.test.test_utils import CustomTestCase

# Global configuration for all tests
DEFAULT_CONFIG = {
    "device": "cuda",
    "dtype": torch.bfloat16,
    "kv_cache_dtype": torch.bfloat16,
    "context_len": 2048,
    "max_bs": 64,
    "tolerance": 1e-2,
    "seed_cache": 42,
    "seed_qkv": 123,
    # MLA model config (TRTLLM MLA has fixed constraints)
    "num_attention_heads": 128,
    "kv_lora_rank": 512,
    "qk_nope_head_dim": 128,
    "qk_rope_head_dim": 64,
    "v_head_dim": 512,
    "num_kv_heads": 1,
    "layer_id": 0,
    "tp_q_head_num": 128,
    "tp_k_head_num": 128,
    "prefill_head_dim": 192,
    "prefill_v_head_dim": 128,
}

ROPE_BASE = 10000
ROPE_SCALING_CONFIG = {
    "beta_fast": 32,
    "beta_slow": 1,
    "factor": 40,
    "mscale": 1.0,
    "mscale_all_dim": 1.0,
    "original_max_position_embeddings": 4096,
    "type": "yarn",
    "rope_type": "deepseek_yarn",
}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `config`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 68-83: Helper routines around build_rotary_emb / 辅助例程
```python
def build_rotary_emb(config, device=None):
    from sglang.srt.layers.rotary_embedding import get_rope_wrapper

    dev = device or config["device"]
    rope_scaling = config.get("rope_scaling", ROPE_SCALING_CONFIG)
    rotary = get_rope_wrapper(
        head_size=config["qk_rope_head_dim"],
        rotary_dim=config["qk_rope_head_dim"],
        max_position=config["context_len"],
        base=ROPE_BASE,
        rope_scaling=rope_scaling,
        is_neox_style=False,
        device=dev,
    )
    rotary.cos_sin_cache = rotary.cos_sin_cache.to(dev)
    return rotary
```
**EN:** This range implements helper routine(s) `build_rotary_emb` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`, `get_rope_wrapper` and `to`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 84-210: Constants and scenario settings / 常量与场景配置
```python


# Centralized test cases for different test scenarios
TEST_CASES = {
    "basic_functionality": [
        {
            "name": "single",
            "batch_size": 1,
            "max_seq_len": 32,
            "page_size": 32,
            "description": "Minimal sanity check",
        },
        {
            "name": "batch",
            "batch_size": 32,
            "max_seq_len": 128,
            "page_size": 32,
            "description": "Medium-scale batch",
        },
    ],
    "output_match": [
        {
            "name": "single_fp16",
            "batch_size": 1,
            "max_seq_len": 64,
            "page_size": 32,
            "description": "Single FP16 vs reference",
        },
        # {
        #     "name": "single_fp8",
        #     "batch_size": 1,
        #     "max_seq_len": 64,
        #     "page_size": 64,
        #     "tolerance": 1e-1,
        #     "kv_cache_dtype": torch.float8_e4m3fn,
        #     "description": "Single FP8 vs reference",
        # },
        {
            "name": "batch_fp16",
            "batch_size": 32,
            "max_seq_len": 64,
            "page_size": 32,
            "description": "Batch FP16 vs reference",
        },
        # {
        #     "name": "batch_fp8",
        #     "batch_size": 32,
        #     "max_seq_len": 64,
        #     "page_size": 64,
        #     "tolerance": 1e-1,
        #     "kv_cache_dtype": torch.float8_e4m3fn,
        #     "description": "Batch FP8 vs reference",
        # },
    ],
    "page_size_consistency": [
        # Only 32 and 64 supported for now in flashinfer TRTLLM-GEN MLA kernel
        {
            "name": "page_32",
            "batch_size": 8,
            "max_seq_len": 128,
            "page_size": 32,
            "description": "32-token pages",
        },
        {
            "name": "page_64",
            "batch_size": 8,
            "max_seq_len": 128,
            "page_size": 64,
            "description": "64-token pages",
        },
    ],
    "shape_sanity_tests": [
        {
            "name": "basic",
            "batch_size": 1,
            "max_seq_len": 128,
            "page_size": 32,
            "description": "Single sequence",
        },
        {
            "name": "basic_different_pagesize",
            "batch_size": 1,
            "max_seq_len": 128,
            "page_size": 64,
            "description": "Different page size",
        },
        {
            "name": "batch",
            "batch_size": 8,
            "max_seq_len": 128,
            "page_size": 32,
            "description": "Batch shapes",
        },
    ],
    "metadata_tests": [
        {
            "name": "single_sequence",
            "batch_size": 1,
            "max_seq_len": 64,
            "page_size": 32,
            "description": "Single sequence metadata",
        },
        {
            "name": "batch_mixed_lengths",
            "batch_size": 8,
            "max_seq_len": 128,
            "page_size": 32,
            "description": "Mixed sequence lengths",
        },
        {
            "name": "large_batch",
            "batch_size": 32,
            "max_seq_len": 256,
            "page_size": 64,
            "description": "Large batch stress test",
        },
        {
            "name": "edge_case_short",
            "batch_size": 4,
            "max_seq_len": 16,
            "page_size": 32,
            "description": "Sub-page sequences",
        },
    ],
}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。

### Lines 211-213: Class definition for MockModelRunner / 类定义
```python
class MockModelRunner:
    """Minimal fake ModelRunner for testing MLA backends."""
```
**EN:** This range declares `MockModelRunner`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 214-235: Helper routines around __init__ / 辅助例程
```python
    def __init__(self, config):
        self.device = config["device"]
        self.dtype = config["dtype"]
        self.kv_cache_dtype = config["kv_cache_dtype"]
        self.page_size = config["page_size"]

        # Server args stub - needed by attention backends
        self.server_args = get_global_server_args()

        # Model-config stub with MLA attributes
        self.model_config = type(
            "ModelConfig",
            (),
            {
                "context_len": config["context_len"],
                "attention_arch": AttentionArch.MLA,
                "num_attention_heads": config["num_attention_heads"],
                "kv_lora_rank": config["kv_lora_rank"],
                "qk_nope_head_dim": config["qk_nope_head_dim"],
                "qk_rope_head_dim": config["qk_rope_head_dim"],
                "v_head_dim": config["v_head_dim"],
                "scaling": 1.0
```
**EN:** This range implements helper routine(s) `__init__` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `get_global_server_args` and `type`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 236-238: Scenario logic / 场景逻辑
```python
                / ((config["qk_nope_head_dim"] + config["qk_rope_head_dim"]) ** 0.5),
                "get_num_kv_heads": staticmethod(lambda _: config["num_kv_heads"]),
            },
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 239-260: Scenario logic / 场景逻辑
```python
        )

        # Req-to-token pool
        max_bs = config["max_bs"]
        max_ctx = self.model_config.context_len
        req_to_token = torch.arange(
            max_bs * max_ctx, dtype=torch.int32, device=self.device
        ).reshape(max_bs, max_ctx)
        self.req_to_token_pool = type(
            "TokenPool",
            (),
            {
                "size": max_bs,
                "req_to_token": req_to_token,
            },
        )

        # KV-token pool (MLA)
        self.token_to_kv_pool = MLATokenToKVPool(
            size=max_bs * max_ctx,
            page_size=config["page_size"],
            dtype=self.kv_cache_dtype,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `arange`, `reshape`, `type` and `pool`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 261-263: Scenario logic / 场景逻辑
```python
            kv_lora_rank=config["kv_lora_rank"],
            qk_rope_head_dim=config["qk_rope_head_dim"],
            layer_num=1,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 264-266: Scenario logic / 场景逻辑
```python
            device=self.device,
            enable_memory_saver=False,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 269-293: Helper routines around compare_outputs / 辅助例程
```python
def compare_outputs(trtllm_out, reference_out, tolerance=1e-2):
    """Compare outputs with detailed analysis."""

    # Basic checks
    assert (
        trtllm_out.shape == reference_out.shape
    ), f"Shape mismatch: {trtllm_out.shape} vs {reference_out.shape}"
    assert (
        trtllm_out.dtype == reference_out.dtype
    ), f"Dtype mismatch: {trtllm_out.dtype} vs {reference_out.dtype}"

    # Check for NaN/Inf
    assert not torch.isnan(trtllm_out).any(), "TRTLLM output contains NaN"
    assert not torch.isnan(reference_out).any(), "Reference output contains NaN"
    assert not torch.isinf(trtllm_out).any(), "TRTLLM output contains Inf"
    assert not torch.isinf(reference_out).any(), "Reference output contains Inf"

    # Element-wise differences
    diff = (trtllm_out - reference_out).abs()
    max_diff = diff.max().item()
    mean_diff = diff.mean().item()

    # Check numerical equivalence
    all_close = torch.allclose(
        trtllm_out, reference_out, rtol=tolerance, atol=tolerance
```
**EN:** This range implements helper routine(s) `compare_outputs` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `assert`, `isnan`, `isinf` and `abs`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 294-312: Scenario logic / 场景逻辑
```python
    )

    if not all_close:
        print(
            f"Comparison failed: max_diff={max_diff:.6f}, mean_diff={mean_diff:.6f}, tolerance={tolerance}"
        )
        # Find top differences for debugging
        flat_diff = diff.flatten()
        top_diff_indices = torch.topk(flat_diff, k=min(5, flat_diff.numel())).indices
        print("Top 5 differences:")
        for i, idx in enumerate(top_diff_indices):
            idx_tuple = np.unravel_index(idx.cpu().numpy(), trtllm_out.shape)
            trt_val = trtllm_out[idx_tuple].item()
            ref_val = reference_out[idx_tuple].item()
            print(
                f"  [{idx_tuple}]: TRTLLM={trt_val:.6f}, Reference={ref_val:.6f}, diff={abs(trt_val-ref_val):.6f}"
            )

    return all_close
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `flatten`, `topk`, `numel` and `unravel_index`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 313-318: Scenario logic / 场景逻辑
```python


@unittest.skipIf(
    not torch.cuda.is_available() or not is_flashinfer_available(),
    "CUDA + flashinfer required",
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipIf`, `is_available` and `is_flashinfer_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 319-322: Class definition for TestTRTLLMMLA / 类定义
```python
class TestTRTLLMMLA(CustomTestCase):
    """Test suite for TRTLLM MLA backend with centralized configuration."""

    @classmethod
```
**EN:** This range declares `TestTRTLLMMLA`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 323-327: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        """Set up global server args for testing."""
        server_args = ServerArgs(model_path="dummy")
        server_args.enable_dp_attention = False
        set_global_server_args_for_scheduler(server_args)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `ServerArgs` and `set_global_server_args_for_scheduler`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 328-329: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 330-331: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        pass
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。

### Lines 333-337: Helper routines around _merge_config / 辅助例程
```python
    def _merge_config(self, test_case):
        """Merge test case with default configuration."""
        config = DEFAULT_CONFIG.copy()
        config.update(test_case)
        return config
```
**EN:** This range implements helper routine(s) `_merge_config` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `copy` and `update`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 339-343: Helper routines around _create_model_components / 辅助例程
```python
    def _create_model_components(self, config, is_prefill=False):
        """Create model runners, backends, and layer for testing."""
        # Create model runners
        model_runner_trtllm = MockModelRunner(config)
        model_runner_reference = MockModelRunner(config)
```
**EN:** This range implements helper routine(s) `_create_model_components` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `MockModelRunner`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 344-363: Scenario logic / 场景逻辑
```python

        # Create backends
        trtllm_backend = TRTLLMMLABackend(model_runner_trtllm)
        reference_backend = FlashInferMLAAttnBackend(model_runner_reference)

        head_dim = (
            config["kv_lora_rank"] + config["qk_rope_head_dim"]
            if not is_prefill
            else config["prefill_head_dim"]
        )
        v_head_dim = (
            config["v_head_dim"] if not is_prefill else config["prefill_v_head_dim"]
        )

        # Create RadixAttention layer
        layer = RadixAttention(
            num_heads=config["num_attention_heads"],
            head_dim=head_dim,
            scaling=model_runner_trtllm.model_config.scaling,
            num_kv_heads=config["num_kv_heads"],
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `TRTLLMMLABackend`, `FlashInferMLAAttnBackend` and `RadixAttention`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 364-368: Scenario logic / 场景逻辑
```python
            layer_id=config["layer_id"],
            v_head_dim=v_head_dim,
            prefix="attn_mqa",
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 369-375: Scenario logic / 场景逻辑
```python
        return (
            model_runner_trtllm,
            model_runner_reference,
            trtllm_backend,
            reference_backend,
            layer,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 377-393: Helper routines around _create_qkv_tensors / 辅助例程
```python
    def _create_qkv_tensors(self, batch_size, config, dtype_override=None):
        """Create Q, K, V random tensors for given batch size with separate MLA components.

        Args:
            batch_size: Batch size.
            config: Configuration dict with model dims and device.
            dtype_override: Optional torch dtype to override config["dtype"].

        Returns:
            Tuple of (q_nope, q_rope, k_nope, k_rope, v, cos_sin_cache)
        """
        device = config["device"]
        target_dtype = dtype_override or config["dtype"]

        # Create separate nope and rope components for Q
        q_nope = torch.randn(
            (batch_size, config["num_attention_heads"], config["kv_lora_rank"]),
```
**EN:** This range implements helper routine(s) `_create_qkv_tensors` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `of` and `randn`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 394-401: Scenario logic / 场景逻辑
```python
            dtype=config["dtype"],
            device=device,
        )
        q_rope = torch.randn(
            (batch_size, config["num_attention_heads"], config["qk_rope_head_dim"]),
            dtype=config["dtype"],
            device=device,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 402-418: Scenario logic / 场景逻辑
```python

        # Create separate nope and rope components for K
        k_nope = torch.randn(
            (batch_size, config["num_kv_heads"], config["kv_lora_rank"]),
            dtype=config["dtype"],
            device=device,
        )
        k_rope = torch.randn(
            (batch_size, config["num_kv_heads"], config["qk_rope_head_dim"]),
            dtype=config["dtype"],
            device=device,
        )

        # V tensor (unchanged)
        v = torch.randn(
            (batch_size, config["num_kv_heads"], config["v_head_dim"]),
            dtype=config["dtype"],
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 419-422: Scenario logic / 场景逻辑
```python
            device=device,
        )

        return q_nope, q_rope, k_nope, k_rope, v
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 424-443: Helper routines around _create_forward_batch / 辅助例程
```python
    def _create_forward_batch(
        self, batch_size, seq_lens, backend, model_runner, config
    ):
        """Create a forward batch for the given backend."""
        fb = ForwardBatch(
            batch_size=batch_size,
            input_ids=torch.randint(0, 100, (batch_size, 1), device=config["device"]),
            out_cache_loc=torch.arange(batch_size, device=config["device"]),
            seq_lens_sum=int(seq_lens.sum().item()),
            forward_mode=ForwardMode.DECODE,
            req_pool_indices=torch.arange(batch_size, device=config["device"]),
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens.cpu(),
            attn_backend=backend,
        )
        fb.req_to_token_pool = model_runner.req_to_token_pool
        fb.token_to_kv_pool = model_runner.token_to_kv_pool

        # Add position information for RoPE
        fb.positions = torch.arange(batch_size, device=config["device"])
```
**EN:** This range implements helper routine(s) `_create_forward_batch` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `ForwardBatch`, `randint`, `arange` and `item`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 444-445: Scenario logic / 场景逻辑
```python

        return fb
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 447-468: Helper routines around _populate_kv_cache / 辅助例程
```python
    def _populate_kv_cache(self, batch_size, seq_lens, model_runners, layer, config):
        """Populate KV cache with identical data for both backends."""
        torch.manual_seed(config["seed_cache"])  # Fixed seed for reproducible cache

        for model_runner in model_runners:
            torch.manual_seed(config["seed_cache"])  # Reset seed for each backend
            for i in range(batch_size):
                seq_len = int(seq_lens[i].item())
                for token_idx in range(seq_len - 1):
                    # Create random K components for MLA
                    cache_k_nope = torch.randn(
                        (1, config["kv_lora_rank"]),
                        dtype=config["dtype"],
                        device=config["device"],
                    )
                    cache_k_rope = torch.randn(
                        (1, config["qk_rope_head_dim"]),
                        dtype=config["dtype"],
                        device=config["device"],
                    )

                    # Calculate cache location
```
**EN:** This range implements helper routine(s) `_populate_kv_cache` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `item` and `randn`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 469-471: Scenario logic / 场景逻辑
```python
                    cache_loc = model_runner.req_to_token_pool.req_to_token[
                        i, token_idx
                    ]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 472-479: Scenario logic / 场景逻辑
```python

                    # Save to KV cache
                    model_runner.token_to_kv_pool.set_mla_kv_buffer(
                        layer,
                        cache_loc.unsqueeze(0),
                        cache_k_nope.squeeze(0),
                        cache_k_rope.squeeze(0),
                    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `set_mla_kv_buffer`, `unsqueeze` and `squeeze`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 481-493: Test routines around test_basic_functionality / 测试例程
```python
    def test_basic_functionality(self):
        """Test basic functionality with minimal setup."""
        print(f"\nRunning basic functionality tests...")

        for test_case in TEST_CASES["basic_functionality"]:
            with self.subTest(test_case=test_case["name"]):
                print(f"  Testing {test_case['name']}: {test_case['description']}")

                config = self._merge_config(test_case)
                batch_size = config["batch_size"]
                max_seq_len = config["max_seq_len"]

                # Create components
```
**EN:** This range defines concrete test routine(s) `test_basic_functionality`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `subTest` and `_merge_config`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 494-505: Scenario logic / 场景逻辑
```python
                model_runner_trtllm, _, trtllm_backend, _, layer = (
                    self._create_model_components(config)
                )

                # Create sequence lengths - properly handle different batch sizes
                if batch_size == 2:
                    seq_lens = torch.tensor(
                        [max_seq_len, max_seq_len // 2], device=config["device"]
                    )
                else:
                    # For larger batch sizes, create varied sequence lengths
                    torch.manual_seed(config["seed_cache"])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_model_components`, `tensor` and `manual_seed`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 506-518: Scenario logic / 场景逻辑
```python
                    seq_lens = torch.randint(
                        max_seq_len // 2,
                        max_seq_len + 1,
                        (batch_size,),
                        device=config["device"],
                    )
                    seq_lens[0] = max_seq_len  # Ensure at least one max length

                # Create forward batch
                fb = self._create_forward_batch(
                    batch_size, seq_lens, trtllm_backend, model_runner_trtllm, config
                )
                trtllm_backend.init_forward_metadata(fb)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint`, `_create_forward_batch` and `init_forward_metadata`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 519-530: Scenario logic / 场景逻辑
```python

                # Populate KV cache
                self._populate_kv_cache(
                    batch_size, seq_lens, [model_runner_trtllm], layer, config
                )

                # Create Q, K, V tensors with separate MLA components
                torch.manual_seed(config["seed_qkv"])
                q_nope, q_rope, k_nope, k_rope, v = self._create_qkv_tensors(
                    batch_size, config
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_populate_kv_cache`, `manual_seed` and `_create_qkv_tensors`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 531-543: Assertions and result checks / 断言与结果检查
```python
                # Run forward decode with separate MLA components
                output = trtllm_backend.forward_decode(
                    q_nope, k_nope, None, layer, fb, q_rope=q_rope, k_rope=k_rope
                )

                # Basic checks
                expected_shape = (
                    batch_size,
                    config["num_attention_heads"] * config["v_head_dim"],
                )
                self.assertEqual(output.shape, expected_shape)
                self.assertEqual(output.dtype, config["dtype"])
                self.assertFalse(torch.isnan(output).any())
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `forward_decode`, `assertEqual`, `assertFalse` and `isnan`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 544-544: Assertions and result checks / 断言与结果检查
```python
                self.assertFalse(torch.isinf(output).any())
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertFalse` and `isinf`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 546-568: Test routines around test_decode_output_match / 测试例程
```python
    def test_decode_output_match(self):
        """Test that TRTLLM and FlashInfer MLA backends produce matching outputs."""
        print(f"\nRunning decode output matching tests...")

        for test_case in TEST_CASES["output_match"]:
            with self.subTest(test_case=test_case["name"]):
                print(f"  Testing {test_case['name']}: {test_case['description']}")

                config = self._merge_config(test_case)
                batch_size = config["batch_size"]
                max_seq_len = config["max_seq_len"]
                use_fp8 = config["kv_cache_dtype"] == torch.float8_e4m3fn

                # Create components
                (
                    model_runner_trtllm,
                    model_runner_reference,
                    trtllm_backend,
                    reference_backend,
                    layer,
                ) = self._create_model_components(config)

                # Create identical sequence lengths for both backends
```
**EN:** This range defines concrete test routine(s) `test_decode_output_match`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `subTest`, `_merge_config`, `components` and `_create_model_components`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 569-570: Scenario logic / 场景逻辑
```python
                torch.manual_seed(config["seed_cache"])
                seq_lens = torch.randint(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `manual_seed` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 571-593: Scenario logic / 场景逻辑
```python
                    1, max_seq_len, (batch_size,), device=config["device"]
                )
                seq_lens[0] = max_seq_len  # Ensure at least one max length

                # Create forward batches with identical inputs
                fb_trtllm = self._create_forward_batch(
                    batch_size,
                    seq_lens.clone(),
                    trtllm_backend,
                    model_runner_trtllm,
                    config,
                )
                fb_reference = self._create_forward_batch(
                    batch_size,
                    seq_lens.clone(),
                    reference_backend,
                    model_runner_reference,
                    config,
                )

                # Initialize metadata for both backends
                trtllm_backend.init_forward_metadata(fb_trtllm)
                reference_backend.init_forward_metadata(fb_reference)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_forward_batch`, `clone` and `init_forward_metadata`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 594-595: Scenario logic / 场景逻辑
```python

                # Populate both KV caches identically
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 596-618: Scenario logic / 场景逻辑
```python
                self._populate_kv_cache(
                    batch_size,
                    seq_lens,
                    [model_runner_trtllm, model_runner_reference],
                    layer,
                    config,
                )

                # Create Q, K, V tensors for current decode step
                torch.manual_seed(config["seed_qkv"])

                q_nope_ref, q_rope_ref, k_nope_ref, k_rope_ref, v_ref = (
                    self._create_qkv_tensors(batch_size, config)
                )
                q_nope_trt, q_rope_trt, k_nope_trt, k_rope_trt, v_trt = (
                    q_nope_ref.clone(),
                    q_rope_ref.clone(),
                    k_nope_ref.clone(),
                    k_rope_ref.clone(),
                    v_ref.clone(),
                )
                tolerance = config["tolerance"]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_populate_kv_cache`, `manual_seed`, `_create_qkv_tensors` and `clone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 619-620: Scenario logic / 场景逻辑
```python
                extra_args = {}
                if use_fp8:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 621-643: Scenario logic / 场景逻辑
```python
                    # TRT kernel applies RoPE + FP8 quantization internally
                    # pre-apply RoPE on the reference (FlashInfer) path here so
                    # both paths share the same rope params/cache while keeping
                    # the TRT path unrotated.
                    rotary_emb = build_rotary_emb(config)
                    q_rope_ref, k_rope_ref = rotary_emb(
                        fb_reference.positions, q_rope_ref, k_rope_ref
                    )
                    extra_args = {
                        "cos_sin_cache": rotary_emb.cos_sin_cache,
                        "is_neox": rotary_emb.is_neox_style,
                    }

                    dtype = q_rope_ref.dtype
                    q_rope_ref = q_rope_ref.to(torch.float8_e4m3fn).to(dtype)
                    q_nope_ref = q_nope_ref.to(torch.float8_e4m3fn).to(dtype)
                    k_rope_ref = k_rope_ref.to(torch.float8_e4m3fn).to(dtype)
                    k_nope_ref = k_nope_ref.to(torch.float8_e4m3fn).to(dtype)

                # Run forward decode on both backends
                out_trtllm = trtllm_backend.forward_decode(
                    q_nope_trt,
                    k_nope_trt,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `reference`, `build_rotary_emb`, `rotary_emb` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 644-645: Scenario logic / 场景逻辑
```python
                    None,
                    layer,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 646-668: Assertions and result checks / 断言与结果检查
```python
                    fb_trtllm,
                    q_rope=q_rope_trt,
                    k_rope=k_rope_trt,
                    **extra_args,
                )

                # Reference backend should also take separate components, not concatenated
                out_reference = reference_backend.forward_decode(
                    q_nope_ref,
                    k_nope_ref,
                    v_ref,
                    layer,
                    fb_reference,
                    q_rope=q_rope_ref,
                    k_rope=k_rope_ref,
                )

                # Compare outputs
                comparison_passed = compare_outputs(
                    out_trtllm, out_reference, tolerance=tolerance
                )

                self.assertTrue(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `forward_decode`, `compare_outputs` and `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 669-670: Scenario logic / 场景逻辑
```python
                    comparison_passed,
                    f"TRTLLM and Reference outputs differ beyond tolerance. "
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 671-673: Scenario logic / 场景逻辑
```python
                    f"Config: {test_case['name']}, "
                    f"Max diff: {(out_trtllm - out_reference).abs().max().item()}",
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `abs` and `item`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 675-693: Test routines around test_page_size_consistency / 测试例程
```python
    def test_page_size_consistency(self):
        """Test output consistency across different page sizes."""
        print(f"\nRunning page size consistency tests...")

        for test_case in TEST_CASES["page_size_consistency"]:
            with self.subTest(test_case=test_case["name"]):
                print(f"  Testing {test_case['name']}: {test_case['description']}")

                config = self._merge_config(test_case)
                batch_size = config["batch_size"]
                max_seq_len = config["max_seq_len"]

                # Create components
                model_runner, _, backend, _, layer = self._create_model_components(
                    config
                )

                # Create sequence lengths
                torch.manual_seed(config["seed_cache"])
```
**EN:** This range defines concrete test routine(s) `test_page_size_consistency`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `subTest`, `_merge_config`, `_create_model_components` and `manual_seed`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 694-699: Scenario logic / 场景逻辑
```python
                seq_lens = torch.randint(
                    1, max_seq_len, (batch_size,), device=config["device"]
                )
                seq_lens[0] = max_seq_len

                # Create forward batch
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 700-718: Scenario logic / 场景逻辑
```python
                fb = self._create_forward_batch(
                    batch_size, seq_lens, backend, model_runner, config
                )
                backend.init_forward_metadata(fb)

                # Populate KV cache
                self._populate_kv_cache(
                    batch_size, seq_lens, [model_runner], layer, config
                )

                # Create Q, K, V tensors with separate MLA components
                torch.manual_seed(config["seed_qkv"])
                q_nope, q_rope, k_nope, k_rope, v = self._create_qkv_tensors(
                    batch_size, config
                )

                # Run forward decode with separate MLA components
                output = backend.forward_decode(
                    q_nope, k_nope, None, layer, fb, q_rope=q_rope, k_rope=k_rope
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_forward_batch`, `init_forward_metadata`, `_populate_kv_cache` and `manual_seed`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 719-724: Scenario logic / 场景逻辑
```python
                )

                expected_shape = (
                    batch_size,
                    config["num_attention_heads"] * config["v_head_dim"],
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 725-731: Assertions and result checks / 断言与结果检查
```python
                self.assertEqual(
                    output.shape,
                    expected_shape,
                    f"Output shape mismatch: {output.shape} vs {expected_shape}",
                )
                self.assertFalse(torch.isnan(output).any(), "Output contains NaN")
                self.assertFalse(torch.isinf(output).any(), "Output contains Inf")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `assertFalse`, `isnan` and `isinf`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 733-743: Test routines around test_shape_sanity / 测试例程
```python
    def test_shape_sanity(self):
        """Check decode shapes across several configurations."""
        print(f"\nRunning shape sanity tests...")

        for test_case in TEST_CASES["shape_sanity_tests"]:
            with self.subTest(test_case=test_case["name"]):
                print(f"  Testing {test_case['name']}: {test_case['description']}")

                config = self._merge_config(test_case)
                batch_size = config["batch_size"]
                max_seq_len = config["max_seq_len"]
```
**EN:** This range defines concrete test routine(s) `test_shape_sanity`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `subTest` and `_merge_config`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 744-757: Scenario logic / 场景逻辑
```python

                model_runner, _, backend, _, layer = self._create_model_components(
                    config
                )

                # Random seq lens (ensure one matches max)
                torch.manual_seed(config["seed_cache"])
                seq_lens = torch.randint(
                    1, max_seq_len, (batch_size,), device=config["device"]
                )
                seq_lens[0] = max_seq_len

                fb = self._create_forward_batch(
                    batch_size, seq_lens, backend, model_runner, config
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_model_components`, `lens`, `manual_seed` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 758-768: Scenario logic / 场景逻辑
```python
                )
                backend.init_forward_metadata(fb)

                # Create Q, K, V tensors with separate MLA components
                torch.manual_seed(config["seed_qkv"])
                q_nope = torch.randn(
                    (batch_size, config["num_attention_heads"], config["kv_lora_rank"]),
                    dtype=config["dtype"],
                    device=config["device"],
                )
                k_nope = torch.randn(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `init_forward_metadata`, `manual_seed` and `randn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 769-782: Scenario logic / 场景逻辑
```python
                    (batch_size, config["num_kv_heads"], config["kv_lora_rank"]),
                    dtype=config["dtype"],
                    device=config["device"],
                )
                q_rope = torch.randn(
                    (
                        batch_size,
                        config["num_attention_heads"],
                        config["qk_rope_head_dim"],
                    ),
                    dtype=config["dtype"],
                    device=config["device"],
                )
                k_rope = torch.randn(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 783-793: Scenario logic / 场景逻辑
```python
                    (batch_size, config["num_kv_heads"], config["qk_rope_head_dim"]),
                    dtype=config["dtype"],
                    device=config["device"],
                )
                v = None  # Test with None v

                # Run forward decode
                output = backend.forward_decode(
                    q_nope, k_nope, v, layer, fb, q_rope=q_rope, k_rope=k_rope
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `forward_decode`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 794-807: Assertions and result checks / 断言与结果检查
```python
                # Shape and sanity checks
                expected_shape = (
                    batch_size,
                    config["num_attention_heads"] * config["v_head_dim"],
                )
                self.assertEqual(
                    output.shape,
                    expected_shape,
                    f"Output shape mismatch for {test_case['name']}",
                )
                self.assertEqual(output.dtype, config["dtype"])
                self.assertEqual(output.device.type, "cuda")
                self.assertFalse(
                    torch.isnan(output).any(),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `assertFalse` and `isnan`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 808-813: Assertions and result checks / 断言与结果检查
```python
                    f"Output contains NaN for {test_case['name']}",
                )
                self.assertFalse(
                    torch.isinf(output).any(),
                    f"Output contains Inf for {test_case['name']}",
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertFalse` and `isinf`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 815-818: Test routines around test_metadata_initialization / 测试例程
```python
    def test_metadata_initialization(self):
        """Test TRTLLM MLA metadata initialization and structure."""
        print(f"\nRunning metadata initialization tests...")
```
**EN:** This range defines concrete test routine(s) `test_metadata_initialization`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 819-839: Scenario logic / 场景逻辑
```python
        for test_case in TEST_CASES["metadata_tests"]:
            with self.subTest(test_case=test_case["name"]):
                print(f"  Testing {test_case['name']}: {test_case['description']}")

                config = self._merge_config(test_case)
                batch_size = config["batch_size"]
                max_seq_len = config["max_seq_len"]

                # Create components
                model_runner, _, backend, _, layer = self._create_model_components(
                    config
                )

                # Create varied sequence lengths
                torch.manual_seed(config["seed_cache"])
                if batch_size == 1:
                    seq_lens = torch.tensor([max_seq_len], device=config["device"])
                else:
                    seq_lens = torch.randint(
                        max(1, max_seq_len // 4),
                        max_seq_len + 1,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `subTest`, `_merge_config`, `_create_model_components` and `manual_seed`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 840-843: Scenario logic / 场景逻辑
```python
                        (batch_size,),
                        device=config["device"],
                    )
                    seq_lens[0] = max_seq_len  # Ensure at least one max length
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 844-864: Assertions and result checks / 断言与结果检查
```python

                # Create forward batch
                fb = self._create_forward_batch(
                    batch_size, seq_lens, backend, model_runner, config
                )

                # Initialize metadata
                backend.init_forward_metadata(fb)

                # Verify metadata exists
                self.assertIsNotNone(backend.forward_decode_metadata)
                self.assertIsInstance(
                    backend.forward_decode_metadata, TRTLLMMLADecodeMetadata
                )

                # Test metadata structure
                metadata = backend.forward_decode_metadata
                self.assertIsNotNone(
                    metadata.block_kv_indices, "Block KV indices should be created"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `_create_forward_batch`, `init_forward_metadata`, `assertIsNotNone` and `assertIsInstance`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 865-868: Assertions and result checks / 断言与结果检查
```python
                # Test block KV indices properties
                self.assertEqual(metadata.block_kv_indices.device.type, "cuda")
                self.assertEqual(metadata.block_kv_indices.dtype, torch.int32)
                self.assertEqual(metadata.block_kv_indices.shape[0], batch_size)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 869-874: Assertions and result checks / 断言与结果检查
```python

                # Verify block indices are valid (>= -1, since -1 is padding)
                self.assertTrue(
                    (metadata.block_kv_indices >= -1).all(),
                    "All block indices should be >= -1 (with -1 as padding)",
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `valid` and `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 876-893: Test routines around test_metadata_block_calculation / 测试例程
```python
    def test_metadata_block_calculation(self):
        """Test block count calculation logic."""
        print(f"\nRunning metadata block calculation tests...")

        test_scenarios = [
            {"seq_len": 31, "page_size": 32, "expected_min_blocks": 1},
            {"seq_len": 32, "page_size": 32, "expected_min_blocks": 1},
            {"seq_len": 33, "page_size": 32, "expected_min_blocks": 2},
            {"seq_len": 128, "page_size": 32, "expected_min_blocks": 4},
            {"seq_len": 128, "page_size": 64, "expected_min_blocks": 2},
        ]

        for scenario in test_scenarios:
            with self.subTest(scenario=scenario):
                config = self._merge_config(
                    {
                        "batch_size": 1,
                        "max_seq_len": scenario["seq_len"],
```
**EN:** This range defines concrete test routine(s) `test_metadata_block_calculation`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `subTest` and `_merge_config`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 894-900: Scenario logic / 场景逻辑
```python
                        "page_size": scenario["page_size"],
                    }
                )

                model_runner, _, backend, _, _ = self._create_model_components(config)

                # Test internal block calculation
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_model_components`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 901-918: Assertions and result checks / 断言与结果检查
```python
                calculated_blocks = backend._calc_padded_blocks(scenario["seq_len"])

                # Should be at least the minimum required
                self.assertGreaterEqual(
                    calculated_blocks,
                    scenario["expected_min_blocks"],
                    f"Calculated blocks ({calculated_blocks}) should be >= minimum required ({scenario['expected_min_blocks']})",
                )

                # Should satisfy page_size constraint
                total_tokens = calculated_blocks * scenario["page_size"]
                self.assertGreaterEqual(
                    total_tokens,
                    scenario["seq_len"],
                    f"Total tokens ({total_tokens}) should cover sequence length ({scenario['seq_len']})",
                )

                # Should satisfy TRT-LLM and Triton constraints
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `_calc_padded_blocks`, `assertGreaterEqual`, `blocks` and `required`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 919-925: Assertions and result checks / 断言与结果检查
```python
                trtllm_constraint = 128 // scenario["page_size"]
                triton_constraint = get_num_page_per_block_flashmla(
                    scenario["page_size"]
                )
                constraint_lcm = math.lcm(trtllm_constraint, triton_constraint)
                self.assertEqual(
                    calculated_blocks % constraint_lcm,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `get_num_page_per_block_flashmla`, `lcm` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 926-928: Scenario logic / 场景逻辑
```python
                    0,
                    f"Block count should be multiple of LCM of constraints ({constraint_lcm})",
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `constraints`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 930-943: Test routines around test_metadata_kv_indices_correctness / 测试例程
```python
    def test_metadata_kv_indices_correctness(self):
        """Test KV indices creation and correctness."""
        print(f"\nRunning KV indices correctness tests...")

        for test_case in TEST_CASES["metadata_tests"][
            :2
        ]:  # Test subset for performance
            with self.subTest(test_case=test_case["name"]):
                print(f"  Testing {test_case['name']}: {test_case['description']}")

                config = self._merge_config(test_case)
                batch_size = config["batch_size"]
                max_seq_len = config["max_seq_len"]
```
**EN:** This range defines concrete test routine(s) `test_metadata_kv_indices_correctness`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `subTest` and `_merge_config`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 944-954: Scenario logic / 场景逻辑
```python
                model_runner, _, backend, _, layer = self._create_model_components(
                    config
                )

                # Create known sequence lengths
                torch.manual_seed(config["seed_cache"])
                if batch_size == 1:
                    seq_lens = torch.tensor([max_seq_len], device=config["device"])
                else:
                    seq_lens = torch.randint(
                        max_seq_len // 2,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_model_components`, `manual_seed`, `tensor` and `randint`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 955-968: Scenario logic / 场景逻辑
```python
                        max_seq_len + 1,
                        (batch_size,),
                        device=config["device"],
                    )

                fb = self._create_forward_batch(
                    batch_size, seq_lens, backend, model_runner, config
                )

                # Populate some KV cache to have valid indices
                self._populate_kv_cache(
                    batch_size, seq_lens, [model_runner], layer, config
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_forward_batch` and `_populate_kv_cache`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 969-979: Scenario logic / 场景逻辑
```python
                # Initialize metadata
                backend.init_forward_metadata(fb)
                metadata = backend.forward_decode_metadata

                # Verify KV indices structure
                block_kv_indices = metadata.block_kv_indices

                for i in range(batch_size):
                    seq_len = seq_lens[i].item()
                    expected_blocks = backend._calc_padded_blocks(seq_len)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `init_forward_metadata`, `item` and `_calc_padded_blocks`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 980-993: Assertions and result checks / 断言与结果检查
```python
                    # Count valid (non -1) indices for this sequence
                    valid_indices = (block_kv_indices[i] >= 0).sum().item()

                    # Should have at least enough blocks for the sequence
                    min_required_blocks = (seq_len + config["page_size"] - 1) // config[
                        "page_size"
                    ]
                    self.assertGreaterEqual(
                        valid_indices,
                        min_required_blocks,
                        f"Sequence {i} should have at least {min_required_blocks} valid blocks, got {valid_indices}",
                    )

                    # Verify indices are within valid range
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `valid`, `item` and `assertGreaterEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 994-1002: Assertions and result checks / 断言与结果检查
```python
                    valid_block_indices = block_kv_indices[i][block_kv_indices[i] >= 0]
                    if len(valid_block_indices) > 0:
                        max_possible_blocks = (
                            model_runner.token_to_kv_pool.size // config["page_size"]
                        )
                        self.assertTrue(
                            (valid_block_indices < max_possible_blocks).all(),
                            f"All block indices should be < {max_possible_blocks}",
                        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1004-1018: Test routines around test_metadata_cuda_graph_compatibility / 测试例程
```python
    def test_metadata_cuda_graph_compatibility(self):
        """Test metadata compatibility with CUDA graph capture/replay."""
        print(f"\nRunning CUDA graph compatibility tests...")

        config = self._merge_config(
            {"batch_size": 4, "max_seq_len": 64, "page_size": 32}
        )

        model_runner, _, backend, _, layer = self._create_model_components(config)
        batch_size = config["batch_size"]

        # Initialize CUDA graph state
        backend.init_cuda_graph_state(
            max_bs=batch_size, max_num_tokens=config["max_seq_len"] * batch_size
        )
```
**EN:** This range defines concrete test routine(s) `test_metadata_cuda_graph_compatibility`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_merge_config`, `_create_model_components` and `init_cuda_graph_state`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1019-1028: Assertions and result checks / 断言与结果检查
```python

        # Verify CUDA graph buffers are allocated
        self.assertIsNotNone(backend.decode_cuda_graph_kv_indices)

        # Test capture metadata
        seq_lens = torch.full(
            (batch_size,), config["max_seq_len"], device=config["device"]
        )
        req_pool_indices = torch.arange(batch_size, device=config["device"])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsNotNone`, `full` and `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1029-1043: Assertions and result checks / 断言与结果检查
```python
        backend.init_forward_metadata_capture_cuda_graph(
            bs=batch_size,
            num_tokens=batch_size,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            encoder_lens=None,
            forward_mode=ForwardMode.DECODE,
            spec_info=None,
        )

        # Verify capture metadata
        self.assertIn(batch_size, backend.decode_cuda_graph_metadata)
        capture_metadata = backend.decode_cuda_graph_metadata[batch_size]

        self.assertIsNotNone(capture_metadata.block_kv_indices)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `init_forward_metadata_capture_cuda_graph`, `assertIn` and `assertIsNotNone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1044-1053: Scenario logic / 场景逻辑
```python

        # Test replay with different sequence lengths
        new_seq_lens = torch.randint(
            config["max_seq_len"] // 2,
            config["max_seq_len"] + 1,
            (batch_size,),
            device=config["device"],
        )

        backend.init_forward_metadata_replay_cuda_graph(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randint` and `init_forward_metadata_replay_cuda_graph`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1054-1066: Assertions and result checks / 断言与结果检查
```python
            bs=batch_size,
            req_pool_indices=req_pool_indices,
            seq_lens=new_seq_lens,
            seq_lens_sum=new_seq_lens.sum().item(),
            encoder_lens=None,
            forward_mode=ForwardMode.DECODE,
            spec_info=None,
            seq_lens_cpu=new_seq_lens.cpu(),
        )

        # Verify replay updated the metadata
        replay_metadata = backend.forward_decode_metadata
        self.assertIsNotNone(replay_metadata)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `item`, `cpu` and `assertIsNotNone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1068-1068: Test routines around test_metadata_consistency_across_calls / 测试例程
```python
    def test_metadata_consistency_across_calls(self):
```
**EN:** This range defines concrete test routine(s) `test_metadata_consistency_across_calls`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 1069-1092: Scenario logic / 场景逻辑
```python
        """Test metadata consistency across multiple forward calls."""
        print(f"\nRunning metadata consistency tests...")

        config = self._merge_config(
            {"batch_size": 2, "max_seq_len": 64, "page_size": 32}
        )

        model_runner, _, backend, _, layer = self._create_model_components(config)

        # First call
        seq_lens_1 = torch.tensor([32, 48], device=config["device"])
        fb_1 = self._create_forward_batch(
            config["batch_size"], seq_lens_1, backend, model_runner, config
        )
        backend.init_forward_metadata(fb_1)
        metadata_1 = backend.forward_decode_metadata

        # Second call with same sequence lengths
        seq_lens_2 = torch.tensor([32, 48], device=config["device"])
        fb_2 = self._create_forward_batch(
            config["batch_size"], seq_lens_2, backend, model_runner, config
        )
        backend.init_forward_metadata(fb_2)
        metadata_2 = backend.forward_decode_metadata
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_merge_config`, `_create_model_components`, `tensor` and `_create_forward_batch`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1094-1109: Assertions and result checks / 断言与结果检查
```python
        # Metadata structure should be consistent
        self.assertEqual(
            metadata_1.block_kv_indices.shape, metadata_2.block_kv_indices.shape
        )

        # Third call with different sequence lengths
        seq_lens_3 = torch.tensor([16, 64], device=config["device"])
        fb_3 = self._create_forward_batch(
            config["batch_size"], seq_lens_3, backend, model_runner, config
        )
        backend.init_forward_metadata(fb_3)
        metadata_3 = backend.forward_decode_metadata

        # Should still have valid structure
        self.assertIsNotNone(metadata_3.block_kv_indices)
        self.assertEqual(metadata_3.block_kv_indices.shape[0], config["batch_size"])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `tensor`, `_create_forward_batch` and `init_forward_metadata`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1111-1118: Test routines around test_prefill_output_match_self_attention / 测试例程
```python
    def test_prefill_output_match_self_attention(self):
        """Test prefill (forward) behavior of TRTLLM MLA backend vs reference."""
        print(f"\nRunning prefill output tests...")

        for test_case in TEST_CASES["output_match"][:2]:  # Just a subset for speed
            with self.subTest(test_case=test_case["name"]):
                print(
                    f"Prefill Testing {test_case['name']}: {test_case['description']}"
```
**EN:** This range defines concrete test routine(s) `test_prefill_output_match_self_attention`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `prefill` and `subTest`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1119-1135: Scenario logic / 场景逻辑
```python
                )

                config = self._merge_config(test_case)
                batch_size = config["batch_size"]
                max_seq_len = config["max_seq_len"]

                # Create components
                (
                    model_runner_trtllm,
                    model_runner_reference,
                    trtllm_backend,
                    reference_backend,
                    layer,
                ) = self._create_model_components(config, is_prefill=True)

                # Prefill uses full sequences
                seq_lens = torch.full(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_merge_config`, `components`, `_create_model_components` and `full`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1136-1143: Helper routines around _create_forward_batch_prefill / 辅助例程
```python
                    (batch_size,), max_seq_len, device=config["device"]
                )

                def _create_forward_batch_prefill(
                    batch_size,
                    seq_lens,
                    extend_prefix_lens,
                    backend,
```
**EN:** This range implements helper routine(s) `_create_forward_batch_prefill` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 1144-1160: Scenario logic / 场景逻辑
```python
                    model_runner,
                    config,
                ):
                    """Create a forward batch for the given backend."""

                    fb = ForwardBatch(
                        batch_size=batch_size,
                        input_ids=torch.randint(
                            0, 100, (batch_size, 1), device=config["device"]
                        ),
                        out_cache_loc=torch.arange(batch_size, device=config["device"]),
                        seq_lens_sum=int(seq_lens.sum().item()),
                        extend_prefix_lens=extend_prefix_lens,
                        extend_prefix_lens_cpu=extend_prefix_lens.cpu().int().tolist(),
                        extend_seq_lens_cpu=(seq_lens - extend_prefix_lens)
                        .cpu()
                        .int()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ForwardBatch`, `randint`, `arange` and `item`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1161-1168: Scenario logic / 场景逻辑
```python
                        .tolist(),
                        forward_mode=ForwardMode.EXTEND,
                        req_pool_indices=torch.arange(
                            batch_size, device=config["device"]
                        ),
                        seq_lens=seq_lens,
                        seq_lens_cpu=seq_lens.cpu(),
                        attn_attend_prefix_cache=False,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tolist`, `arange` and `cpu`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1169-1185: Scenario logic / 场景逻辑
```python
                        mha_return_lse=False,
                        attn_backend=backend,
                    )
                    fb.req_to_token_pool = model_runner.req_to_token_pool
                    fb.token_to_kv_pool = model_runner.token_to_kv_pool

                    # Add position information for RoPE
                    fb.positions = torch.arange(batch_size, device=config["device"])

                    return fb

                # Create forward batches
                fb_trtllm = _create_forward_batch_prefill(
                    batch_size,
                    seq_lens.clone(),
                    torch.zeros(batch_size, device=config["device"], dtype=torch.int32),
                    trtllm_backend,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `arange`, `_create_forward_batch_prefill`, `clone` and `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1186-1193: Scenario logic / 场景逻辑
```python
                    model_runner_trtllm,
                    config,
                )
                fb_reference = _create_forward_batch_prefill(
                    batch_size,
                    seq_lens.clone(),
                    torch.zeros(batch_size, device=config["device"], dtype=torch.int32),
                    reference_backend,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_forward_batch_prefill`, `clone` and `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1194-1210: Helper routines around _create_qkv_tensors_prefill / 辅助例程
```python
                    model_runner_reference,
                    config,
                )

                # Initialize metadata for both backends
                trtllm_backend.init_forward_metadata(fb_trtllm)
                reference_backend.init_forward_metadata(fb_reference)

                # Create Q, K, V tensors for prefill
                torch.manual_seed(config["seed_qkv"])

                def _create_qkv_tensors_prefill(
                    batch_size, seq_len, config, dtype_override=None
                ):
                    """Create Q, K, V tensors for prefill, using config for head_num and head_dim."""
                    device = config["device"]
                    dtype = dtype_override or config["dtype"]
```
**EN:** This range implements helper routine(s) `_create_qkv_tensors_prefill` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `init_forward_metadata` and `manual_seed`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1211-1218: Scenario logic / 场景逻辑
```python

                    total_tokens = batch_size * seq_len

                    tp_q_head_num = config["tp_q_head_num"]
                    tp_k_head_num = config["tp_k_head_num"]
                    head_dim = config["prefill_head_dim"]
                    v_head_dim = config["prefill_v_head_dim"]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 1219-1235: Scenario logic / 场景逻辑
```python
                    q = torch.randn(
                        (total_tokens, tp_q_head_num * head_dim),
                        dtype=dtype,
                        device=device,
                    )
                    k = torch.randn(
                        (total_tokens, tp_k_head_num * head_dim),
                        dtype=dtype,
                        device=device,
                    )
                    v = torch.randn(
                        (total_tokens, tp_k_head_num * v_head_dim),
                        dtype=dtype,
                        device=device,
                    )

                    # Reshape as requested
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1236-1243: Scenario logic / 场景逻辑
```python
                    q = q.view(-1, tp_q_head_num, head_dim)
                    k = k.view(-1, tp_k_head_num, head_dim)
                    v = v.view(-1, tp_k_head_num, v_head_dim)

                    return q, k, v

                q, k, v = _create_qkv_tensors_prefill(batch_size, max_seq_len, config)
                # Run prefill on both backends
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `view` and `_create_qkv_tensors_prefill`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1244-1260: Assertions and result checks / 断言与结果检查
```python
                out_trtllm = trtllm_backend.forward_extend(
                    q, k, v, layer, fb_trtllm, False
                ).view(-1, layer.tp_q_head_num * layer.v_head_dim)
                out_reference = reference_backend.forward_extend(
                    q, k, v, layer, fb_reference, False
                )

                tolerance = config.get("tolerance", 1e-2)
                comparison_passed = compare_outputs(
                    out_trtllm, out_reference, tolerance=tolerance
                )
                self.assertTrue(
                    comparison_passed,
                    f"TRTLLM and Reference prefill outputs differ beyond tolerance. "
                    f"Config: {test_case['name']}, "
                    f"Max diff: {(out_trtllm - out_reference).abs().max().item()}",
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `forward_extend`, `view`, `get` and `compare_outputs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1262-1268: Test routines around test_draft_extend_padding_unpadding_kernels / 测试例程
```python
    def test_draft_extend_padding_unpadding_kernels(self):
        """Test TRTLLM MLA Triton kernels: pad_draft_extend_query_kernel and unpad_draft_extend_output_kernel."""

        # Import the kernels
        from sglang.srt.layers.attention.trtllm_mla_backend import (
            pad_draft_extend_query_kernel,
            unpad_draft_extend_output_kernel,
```
**EN:** This range defines concrete test routine(s) `test_draft_extend_padding_unpadding_kernels`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 1269-1286: Helper routines around _create_test_data / 辅助例程
```python
        )

        def _create_test_data(
            self, batch_size, max_seq_len, num_heads, head_dim, dtype=torch.float32
        ):
            """Create test data for kernel testing."""
            device = torch.device("cuda")

            # Create sequence lengths (varying lengths for each batch)
            seq_lens = torch.randint(
                1, max_seq_len + 1, (batch_size,), device=device, dtype=torch.int32
            )

            # Create cumulative sequence lengths
            cum_seq_lens = torch.zeros(batch_size + 1, device=device, dtype=torch.int32)
            cum_seq_lens[1:] = torch.cumsum(seq_lens, dim=0)

            # Create input query tensor (flattened format)
```
**EN:** This range implements helper routine(s) `_create_test_data` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `device`, `lengths`, `randint` and `zeros`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1287-1293: Scenario logic / 场景逻辑
```python
            total_tokens = cum_seq_lens[-1].item()
            q_input = torch.randn(
                total_tokens, num_heads, head_dim, device=device, dtype=dtype
            )

            # Create padded query tensor (batch format)
            padded_q = torch.zeros(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `item`, `randn`, `tensor` and `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1294-1311: Helper routines around _create_test_output_data / 辅助例程
```python
                batch_size, max_seq_len, num_heads, head_dim, device=device, dtype=dtype
            )

            return q_input, padded_q, seq_lens, cum_seq_lens

        def _create_test_output_data(
            self,
            batch_size,
            token_per_batch,
            tp_q_head_num,
            v_head_dim,
            dtype=torch.float32,
        ):
            """Create test data for unpad kernel testing."""
            device = torch.device("cuda")

            # Create accept lengths (varying lengths for each batch)
            num_accept_tokens_per_req = torch.randint(
```
**EN:** This range implements helper routine(s) `_create_test_output_data` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `device`, `lengths` and `randint`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1312-1318: Scenario logic / 场景逻辑
```python
                1, token_per_batch + 1, (batch_size,), device=device, dtype=torch.int32
            )

            # Create cumulative accept lengths
            cum_accept_lengths = torch.zeros(
                batch_size + 1, device=device, dtype=torch.int32
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1319-1336: Scenario logic / 场景逻辑
```python
            cum_accept_lengths[1:] = torch.cumsum(num_accept_tokens_per_req, dim=0)

            # Create raw output tensor (batch format)
            raw_out = torch.randn(
                batch_size,
                token_per_batch,
                tp_q_head_num,
                v_head_dim,
                device=device,
                dtype=dtype,
            )

            # Create output tensor (flattened format)
            total_tokens = cum_accept_lengths[-1].item()
            output = torch.empty(
                total_tokens, tp_q_head_num, v_head_dim, device=device, dtype=dtype
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `cumsum`, `tensor`, `randn` and `item`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1337-1343: Scenario logic / 场景逻辑
```python
            return raw_out, output, num_accept_tokens_per_req, cum_accept_lengths

        # Test 1: pad_draft_extend_query_kernel basic functionality
        with self.subTest(test="pad_kernel_basic"):
            batch_size = 4
            max_seq_len = 8
            num_heads = 16
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `subTest`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1344-1361: Constants and scenario settings / 常量与场景配置
```python
            head_dim = 64

            q_input, padded_q, seq_lens, cum_seq_lens = _create_test_data(
                self, batch_size, max_seq_len, num_heads, head_dim
            )

            # Launch kernel
            BLOCK_SIZE = 64
            grid = (batch_size * max_seq_len,)

            pad_draft_extend_query_kernel[grid](
                q_ptr=q_input,
                padded_q_ptr=padded_q,
                seq_lens_q_ptr=seq_lens,
                cumsum_ptr=cum_seq_lens,
                batch_size=batch_size,
                max_seq_len=max_seq_len,
                num_heads=num_heads,
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `_create_test_data`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1362-1368: Constants and scenario settings / 常量与场景配置
```python
                head_dim=head_dim,
                BLOCK_SIZE=BLOCK_SIZE,
            )

            # Verify the padding worked correctly
            for i in range(batch_size):
                seq_len = seq_lens[i].item()
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `item`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1369-1386: Assertions and result checks / 断言与结果检查
```python

                # Check that valid positions are copied correctly
                for pos in range(seq_len):
                    input_start = cum_seq_lens[i].item()
                    input_pos = input_start + pos

                    # Compare input and output for valid positions
                    input_data = q_input[input_pos]
                    output_data = padded_q[i, pos]

                    torch.testing.assert_close(
                        input_data, output_data, rtol=1e-5, atol=1e-6
                    )

                # Check that invalid positions are zero
                for pos in range(seq_len, max_seq_len):
                    output_data = padded_q[i, pos]
                    self.assertTrue(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `item`, `assert_close` and `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1387-1393: Scenario logic / 场景逻辑
```python
                        torch.allclose(output_data, torch.zeros_like(output_data)),
                        f"Position {pos} in batch {i} should be zero",
                    )

        # Test 2: unpad_draft_extend_output_kernel basic functionality
        with self.subTest(test="unpad_kernel_basic"):
            batch_size = 4
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `allclose`, `zeros_like` and `subTest`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1394-1411: Constants and scenario settings / 常量与场景配置
```python
            token_per_batch = 8
            tp_q_head_num = 16
            v_head_dim = 64

            raw_out, output, num_accept_tokens_per_req, cum_accept_lengths = (
                _create_test_output_data(
                    self, batch_size, token_per_batch, tp_q_head_num, v_head_dim
                )
            )

            # Launch kernel
            BLOCK_SIZE = 64
            grid = (batch_size * token_per_batch,)

            unpad_draft_extend_output_kernel[grid](
                raw_out_ptr=raw_out,
                output_ptr=output,
                num_accept_tokens_ptr=num_accept_tokens_per_req,
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `_create_test_output_data`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1412-1418: Constants and scenario settings / 常量与场景配置
```python
                cumsum_ptr=cum_accept_lengths,
                batch_size=batch_size,
                token_per_batch=token_per_batch,
                tp_q_head_num=tp_q_head_num,
                v_head_dim=v_head_dim,
                BLOCK_SIZE=BLOCK_SIZE,
            )
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。

### Lines 1419-1432: Assertions and result checks / 断言与结果检查
```python

            # Verify the unpadding worked correctly
            for i in range(batch_size):
                accept_len = num_accept_tokens_per_req[i].item()
                output_start = cum_accept_lengths[i].item()

                # Check that valid positions are copied correctly
                for pos in range(accept_len):
                    input_data = raw_out[i, pos]
                    output_data = output[output_start + pos]

                    torch.testing.assert_close(
                        input_data, output_data, rtol=1e-5, atol=1e-6
                    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `item` and `assert_close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 1433-1436: Script entry point / 脚本入口
```python


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
- Token-level inspection / Token 级分析
- LoRA adaptation / LoRA 适配

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `math`, `unittest`
- **Third-party / 第三方库**: `numpy`, `torch`
- **Project Modules / 项目模块**: `sglang.srt.configs.model_config`, `sglang.srt.layers`, `sglang.srt.layers.attention.flashinfer_mla_backend`, `sglang.srt.layers.attention.trtllm_mla_backend`, `sglang.srt.layers.attention.utils`, `sglang.srt.layers.radix_attention`, `sglang.srt.layers.rotary_embedding`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.test.test_utils`
