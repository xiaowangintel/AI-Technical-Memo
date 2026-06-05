# test_flashattn_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/attention/test_flashattn_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `flashattn backend` scenario in `test/manual/attention`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual/attention` 中的 `flashattn backend` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

import torch

from sglang.srt.configs.model_config import AttentionArch
from sglang.srt.layers.attention.flashattention_backend import (
    FlashAttentionBackend,
    draft_decode_set_expand_metadata,
)
from sglang.srt.layers.attention.torch_native_backend import TorchNativeAttnBackend
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.mem_cache.memory_pool import MHATokenToKVPool
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range imports `unittest`, `torch`, `sglang.srt.configs.model_config` and `sglang.srt.layers.attention.flashattention_backend`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 17-17: Class definition for MockModelRunner / 类定义
```python
class MockModelRunner:
```
**EN:** This range declares `MockModelRunner`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 18-41: Helper routines around __init__ / 辅助例程
```python
    def __init__(
        self,
        page_size=1,
        num_heads=2,
        head_dim=8,
    ):
        self.device = "cuda"
        self.dtype = torch.float16
        self.kv_cache_dtype = torch.float16
        self.is_hybrid_swa = False
        self.attention_chunk_size = None
        attention_arch = AttentionArch.MHA
        # Max batch size for the test.
        max_batch_size = 160
        # Total tokens(prefix + extend + decode) in the test should not exceed this length.
        max_context_len = 2048
        self.model_config = type(
            "ModelConfig",
            (),
            {
                "context_len": max_context_len,
                "is_multimodal": False,
                "attention_arch": attention_arch,
                "is_encoder_decoder": False,
```
**EN:** This range implements helper routine(s) `__init__` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `tokens` and `type`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 42-42: Scenario logic / 场景逻辑
```python
                "is_local_attention_model": False,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 43-66: Scenario logic / 场景逻辑
```python
            },
        )()
        self.sliding_window_size = None
        self.kv_cache_dtype = (
            self.dtype
        )  # torch dtype, required by FlashAttentionBackend

        # server_args is still needed for string-based config (kv_cache_dtype_str)
        self.server_args = type(
            "ServerArgs",
            (),
            {
                "kv_cache_dtype": "auto",  # string version for kv_cache_dtype_str
                "speculative_eagle_topk": None,
                "speculative_num_draft_tokens": 0,
                "enable_deterministic_inference": False,
            },
        )
        self.attn_cp_size = 1
        # Create a large enough req_to_token_pool to fit the test usage.
        self.req_to_token_pool = type(
            "TokenPool",
            (),
            {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `config` and `type`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 67-67: Scenario logic / 场景逻辑
```python
                # A typical max_bs * max_context_len for cuda graph decode
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 68-89: Scenario logic / 场景逻辑
```python
                "size": max_batch_size,
                # Add req_to_token attribute
                "req_to_token": torch.zeros(
                    max_batch_size,
                    max_context_len,
                    dtype=torch.int32,
                    device=self.device,
                ),
            },
        )()
        self.page_size = page_size
        max_total_num_tokens = max_batch_size * max_context_len
        self.token_to_kv_pool = MHATokenToKVPool(
            size=max_total_num_tokens,
            page_size=page_size,
            dtype=self.dtype,
            head_num=num_heads,
            head_dim=head_dim,
            layer_num=1,  # only consider layer=1 for unit test
            device=self.device,
            enable_memory_saver=False,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros` and `MHATokenToKVPool`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-92: Scenario logic / 场景逻辑
```python


@unittest.skipIf(not torch.cuda.is_available(), "Test requires CUDA")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipIf` and `is_available`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 93-93: Class definition for TestFlashAttentionBackend / 类定义
```python
class TestFlashAttentionBackend(CustomTestCase):
```
**EN:** This range declares `TestFlashAttentionBackend`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 94-101: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUp(self):
        # Test parameters
        self.batch_size = 2
        self.seq_len = 256
        self.num_heads = 2
        self.head_dim = 8
        self.device = "cuda"
        self.dtype = torch.float16
```
**EN:** This range implements lifecycle helper(s) `setUp` to prepare or clean up shared resources across the test flow.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。

### Lines 103-111: Helper routines around _init_model_runner / 辅助例程
```python
    def _init_model_runner(self, page_size=1):
        self.model_runner = MockModelRunner(
            page_size=page_size,
            num_heads=self.num_heads,
            head_dim=self.head_dim,
        )
        self.backend = FlashAttentionBackend(self.model_runner)
        self.ref_backend = TorchNativeAttnBackend(self.model_runner)
        self.model_runner.model_config.num_attention_heads = self.num_heads
```
**EN:** This range implements helper routine(s) `_init_model_runner` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `MockModelRunner`, `FlashAttentionBackend` and `TorchNativeAttnBackend`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 113-117: Helper routines around _mock_write_to_req_to_token_pool / 辅助例程
```python
    def _mock_write_to_req_to_token_pool(self, batch_size, seq_len, page_size):
        # if page_size > 1, the token pool stores the index to the page.
        # so we need to multiply the index by page_size.
        self.req_to_token = (
            torch.arange(0, batch_size, dtype=torch.int32, device=self.device)[:, None]
```
**EN:** This range implements helper routine(s) `_mock_write_to_req_to_token_pool` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `arange`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 118-124: Scenario logic / 场景逻辑
```python
            * seq_len
            + torch.arange(0, seq_len, dtype=torch.int32, device=self.device)[None, :]
            + page_size
        )
        self.model_runner.req_to_token_pool.req_to_token[:batch_size, :seq_len] = (
            self.req_to_token
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 126-134: Helper routines around _create_attention_layer / 辅助例程
```python
    def _create_attention_layer(self):
        """Create attention layer for testing."""
        return RadixAttention(
            num_heads=self.num_heads,
            head_dim=self.head_dim,
            scaling=1.0,
            num_kv_heads=self.num_heads,
            layer_id=0,
        )
```
**EN:** This range implements helper routine(s) `_create_attention_layer` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `RadixAttention`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 136-142: Helper routines around _create_qkv_tensors / 辅助例程
```python
    def _create_qkv_tensors(self, tokens_len):
        """Create q, k, v tensors for testing."""
        shape = (tokens_len, self.num_heads, self.head_dim)
        return (
            torch.randn(shape, dtype=self.dtype, device=self.device),
            torch.randn(shape, dtype=self.dtype, device=self.device),
            torch.randn(shape, dtype=self.dtype, device=self.device),
```
**EN:** This range implements helper routine(s) `_create_qkv_tensors` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `randn`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 143-143: Scenario logic / 场景逻辑
```python
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 145-153: Helper routines around _run_reference_forward / 辅助例程
```python
    def _run_reference_forward(
        self, mode, q, k, v, layer, forward_batch, expected_shape
    ):
        """Run reference forward pass using native backend."""
        if mode == ForwardMode.EXTEND:
            output = self.ref_backend.forward_extend(q, k, v, layer, forward_batch)
        else:  # ForwardMode.DECODE
            output = self.ref_backend.forward_decode(q, k, v, layer, forward_batch)
        return output.view(expected_shape)
```
**EN:** This range implements helper routine(s) `_run_reference_forward` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `forward_extend`, `forward_decode` and `view`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 155-167: Helper routines around _verify_output / 辅助例程
```python
    def _verify_output(self, output, expected_shape, output_ref=None):
        """Verify output tensor shape, dtype, and values."""
        self.assertEqual(
            output.shape,
            expected_shape,
            f"Expected shape {expected_shape}, got {output.shape}",
        )
        self.assertEqual(output.dtype, self.dtype)
        self.assertEqual(output.device.type, "cuda")
        self.assertEqual(
            torch.isnan(output).sum().item(), 0, "Output contains NaN values"
        )
```
**EN:** This range implements helper routine(s) `_verify_output` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `isnan` and `item`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 168-179: Scenario logic / 场景逻辑
```python
        if output_ref is not None:
            if not torch.allclose(output, output_ref, atol=1e-1, rtol=0.0):
                # Check where the values differ beyond the given tolerances
                diff_mask = ~torch.isclose(output, output_ref, atol=1e-1, rtol=0.0)

                # Find the first index where the difference occurs
                if diff_mask.any():
                    first_mismatch_idx = diff_mask.nonzero()[0]
                    print(
                        "First mismatch at index:", tuple(first_mismatch_idx.tolist())
                    )
                    print("output:", output[tuple(first_mismatch_idx.tolist())])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `allclose`, `isclose`, `nonzero` and `tolist`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 180-183: Assertions and result checks / 断言与结果检查
```python
                    print("output_ref:", output_ref[tuple(first_mismatch_idx.tolist())])
                raise AssertionError(
                    "Attention output is not close to the torch native backend output"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tolist` and `AssertionError`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 185-192: Helper routines around _create_forward_batch / 辅助例程
```python
    def _create_forward_batch(
        self, mode, q_len=None, prefix_len=0, page_size=1, attn_cp_size=1
    ):
        """Create a forward batch for testing based on mode and lengths."""
        self._init_model_runner(page_size=page_size)

        # Default to self.seq_len if not specified
        q_len = q_len or self.seq_len
```
**EN:** This range implements helper routine(s) `_create_forward_batch` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `_init_model_runner`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 193-209: Scenario logic / 场景逻辑
```python

        if mode == ForwardMode.EXTEND:
            total_len = prefix_len + q_len
            out_cache_start = prefix_len * self.batch_size
            out_cache_end = total_len * self.batch_size

            forward_batch = ForwardBatch(
                batch_size=self.batch_size,
                input_ids=torch.randint(
                    0, 100, (self.batch_size, q_len), device=self.device
                ),
                out_cache_loc=torch.arange(
                    out_cache_start, out_cache_end, device=self.device
                ),
                seq_lens_sum=self.batch_size * total_len,
                forward_mode=mode,
                req_pool_indices=torch.arange(self.batch_size, device=self.device),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ForwardBatch`, `randint` and `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 210-217: Scenario logic / 场景逻辑
```python
                seq_lens=torch.tensor(
                    [total_len] * self.batch_size, device=self.device
                ),
                seq_lens_cpu=torch.tensor([total_len] * self.batch_size, device="cpu"),
                extend_prefix_lens=torch.tensor(
                    [prefix_len] * self.batch_size, device=self.device
                ),
                extend_prefix_lens_cpu=torch.tensor(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 218-234: Scenario logic / 场景逻辑
```python
                    [prefix_len] * self.batch_size, device="cpu"
                ),
                extend_seq_lens=torch.tensor(
                    [q_len] * self.batch_size, device=self.device
                ),
                extend_seq_lens_cpu=torch.tensor(
                    [q_len] * self.batch_size, device="cpu"
                ),
                attn_backend=self.backend,
            )
            if attn_cp_size > 1:
                forward_batch.attn_cp_metadata = type(
                    "AttnCPMetadata",
                    (),
                    {
                        "kv_len_prev_tensor": torch.tensor(
                            [q_len // 2] * self.batch_size,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor` and `type`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 235-242: Scenario logic / 场景逻辑
```python
                            dtype=torch.int32,
                            device=self.device,
                        ),
                        "kv_len_next_tensor": torch.tensor(
                            [q_len] * self.batch_size,
                            dtype=torch.int32,
                            device=self.device,
                        ),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 243-259: Scenario logic / 场景逻辑
```python
                        "actual_seq_q_prev": q_len // 2,
                        "actual_seq_q_next": q_len // 2,
                    },
                )
        else:  # ForwardMode.DECODE
            decode_len = q_len  # Assuming 1 for decode testing
            total_len = self.seq_len + decode_len
            if mode == ForwardMode.DECODE and page_size > 1:
                # Get next page_size multiple of self.seq_len
                out_cache_start = (
                    self.batch_size * self.seq_len // page_size + 1
                ) * page_size
                # out_cache_end is the start of the next block
                out_cache_end = out_cache_start + decode_len * page_size
            else:
                out_cache_start = self.batch_size * self.seq_len
                out_cache_end = self.batch_size * total_len
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 260-267: Scenario logic / 场景逻辑
```python

            forward_batch = ForwardBatch(
                batch_size=self.batch_size,
                input_ids=torch.randint(
                    0, 100, (self.batch_size, decode_len), device=self.device
                ),
                out_cache_loc=torch.tensor(
                    [out_cache_start, out_cache_end], device=self.device
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ForwardBatch`, `randint` and `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 268-284: Scenario logic / 场景逻辑
```python
                ),
                seq_lens_sum=self.batch_size * total_len,
                forward_mode=mode,
                req_pool_indices=torch.arange(self.batch_size, device=self.device),
                seq_lens=torch.tensor(
                    [total_len] * self.batch_size, device=self.device
                ),
                seq_lens_cpu=torch.tensor([total_len] * self.batch_size, device="cpu"),
                attn_backend=self.backend,
            )

        # Add token pool
        forward_batch.req_to_token_pool = self.model_runner.req_to_token_pool

        # Write current batch's req_to_token to req_to_token_pool
        self._mock_write_to_req_to_token_pool(self.batch_size, total_len, page_size)
        # Add kv pool for this forward batch
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `arange`, `tensor` and `_mock_write_to_req_to_token_pool`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 285-287: Scenario logic / 场景逻辑
```python
        forward_batch.token_to_kv_pool = self.model_runner.token_to_kv_pool

        return forward_batch
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 289-292: Helper routines around _setup_kv_cache / 辅助例程
```python
    def _setup_kv_cache(self, forward_batch, layer, cache_len):
        # Create constant values for the prefix cache for easy debugging
        cache_k = torch.ones(
            self.batch_size * cache_len,
```
**EN:** This range implements helper routine(s) `_setup_kv_cache` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `ones`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 293-313: Scenario logic / 场景逻辑
```python
            self.num_heads,
            self.head_dim,
            dtype=self.dtype,
            device=self.device,
        )
        cache_v = (
            torch.ones(
                self.batch_size * cache_len,
                self.num_heads,
                self.head_dim,
                dtype=self.dtype,
                device=self.device,
            )
            * 2
        )

        # Set the prefix KV cache
        forward_batch.token_to_kv_pool.set_kv_buffer(
            layer,
            torch.arange(self.batch_size * cache_len, device=self.device),
            cache_k,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ones`, `set_kv_buffer` and `arange`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 314-317: Scenario logic / 场景逻辑
```python
            cache_v,
            layer.k_scale,
            layer.v_scale,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 319-342: Helper routines around _run_attention_test / 辅助例程
```python
    def _run_attention_test(self, mode, q_len, prefix_len=0, page_size=1):
        """
            Run an attention test with the specified parameters.
        Args:
            mode: ForwardMode.EXTEND or ForwardMode.DECODE
            q_len: Length of the query sequence. For decode mode, q_len is 1.
            prefix_len: Length of the prefix sequence for extend mode
            page_size: Page size for the KV cache
        """
        layer = self._create_attention_layer()

        # Create forward batch and set up
        forward_batch = self._create_forward_batch(mode, q_len, prefix_len, page_size)

        # Create QKV tensors for the input
        q, k, v = self._create_qkv_tensors(self.batch_size * q_len)

        # KV cache for prefixed extend is prefix_len
        # KV cache for decode is same as seq_len
        # No KV cache for extend without prefix
        if mode == ForwardMode.EXTEND:
            if prefix_len > 0:
                self._setup_kv_cache(forward_batch, layer, prefix_len)
        else:
```
**EN:** This range implements helper routine(s) `_run_attention_test` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `_create_attention_layer`, `_create_forward_batch`, `_create_qkv_tensors` and `_setup_kv_cache`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 343-343: Scenario logic / 场景逻辑
```python
            self._setup_kv_cache(forward_batch, layer, self.seq_len)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_setup_kv_cache`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 344-363: Scenario logic / 场景逻辑
```python

        self.backend.init_forward_metadata(forward_batch)

        if mode == ForwardMode.EXTEND:
            expected_shape = (
                self.batch_size * q_len,
                self.num_heads * self.head_dim,
            )
            output = self.backend.forward_extend(q, k, v, layer, forward_batch)
        else:
            expected_shape = (self.batch_size, self.num_heads * self.head_dim)
            output = self.backend.forward_decode(q, k, v, layer, forward_batch)

        output_ref = self._run_reference_forward(
            mode, q, k, v, layer, forward_batch, expected_shape
        )

        self._verify_output(output, expected_shape, output_ref)

        return output
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `init_forward_metadata`, `forward_extend`, `forward_decode` and `_run_reference_forward`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 365-367: Helper routines around _run_attention_cp_test / 辅助例程
```python
    def _run_attention_cp_test(self, mode, q_len, prefix_len=0, page_size=1):
        layer = self._create_attention_layer()
```
**EN:** This range implements helper routine(s) `_run_attention_cp_test` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `_create_attention_layer`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 368-389: Scenario logic / 场景逻辑
```python
        # Create forward batch and set up
        forward_batch = self._create_forward_batch(
            mode, q_len, prefix_len, page_size, attn_cp_size=2
        )
        self.backend.attn_cp_size = 2

        # Create QKV tensors for the input
        q, k, v = self._create_qkv_tensors(self.batch_size * q_len)

        # KV cache for prefixed extend is prefix_len
        # KV cache for decode is same as seq_len
        # No KV cache for extend without prefix
        # Setup KV cache for CP testing - need KV cache to have actual values
        # For extend with CP, we need KV cache populated so attention has something to attend to
        self._setup_kv_cache(forward_batch, layer, q_len)

        self.backend.init_forward_metadata(forward_batch)

        # if mode == ForwardMode.EXTEND:
        expected_shape = (
            self.batch_size * q_len,
            self.num_heads * self.head_dim,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_create_forward_batch`, `_create_qkv_tensors`, `_setup_kv_cache` and `init_forward_metadata`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 390-392: Scenario logic / 场景逻辑
```python
        )

        output = self.backend.forward_extend(q, k, v, layer, forward_batch)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `forward_extend`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 393-403: Scenario logic / 场景逻辑
```python
        # else:
        #     expected_shape = (self.batch_size, self.num_heads * self.head_dim)
        #     output = self.backend.forward_decode(q, k, v, layer, forward_batch)

        output_ref = self._run_reference_forward(
            mode, q, k, v, layer, forward_batch, expected_shape
        )

        self._verify_output(output, expected_shape, output_ref)

        return output
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `forward_decode`, `_run_reference_forward` and `_verify_output`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 405-407: Test routines around test_forward_extend_cp / 测试例程
```python
    def test_forward_extend_cp(self):
        """Test the standard extend operation with context parallel."""
        self._run_attention_cp_test(ForwardMode.EXTEND, q_len=self.seq_len)
```
**EN:** This range defines concrete test routine(s) `test_forward_extend_cp`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_run_attention_cp_test`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 408-439: Scenario logic / 场景逻辑
```python

    # def test_forward_extend_cp_with_prefix(self):
    #     """Test the standard extend operation with context parallel and prefix."""
    #     prefix_len = self.seq_len // 2
    #     extend_len = self.seq_len - prefix_len
    #     self._run_attention_cp_test(ForwardMode.EXTEND, q_len=extend_len, prefix_len=prefix_len)

    # def test_forward_extend(self):
    #     """Test the standard extend operation."""
    #     self._run_attention_test(ForwardMode.EXTEND, q_len=self.seq_len)

    # def test_forward_decode(self):
    #     """Test the decode operation with cached tokens."""
    #     self._run_attention_test(ForwardMode.DECODE, q_len=1)

    # def test_forward_extend_with_prefix(self):
    #     """Test extending from cached prefix tokens."""
    #     prefix_len = self.seq_len // 2
    #     extend_len = self.seq_len - prefix_len
    #     self._run_attention_test(
    #         ForwardMode.EXTEND, q_len=extend_len, prefix_len=prefix_len
    #     )

    # def test_forward_extend_with_page_size_greater_than_1(self):
    #     """Test extending from cached prefix tokens with page size greater than 1."""
    #     self._run_attention_test(ForwardMode.EXTEND, q_len=self.seq_len, page_size=64)

    # def test_forward_decode_with_page_size_greater_than_1(self):
    #     """Test decode operation with page size greater than 1."""
    #     self._run_attention_test(ForwardMode.DECODE, q_len=1, page_size=64)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `test_forward_extend_cp_with_prefix`, `_run_attention_cp_test`, `test_forward_extend` and `_run_attention_test`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 440-445: Class definition for TestUpdateDraftDecodeSetExpandMetadata / 类定义
```python
class TestUpdateDraftDecodeSetExpandMetadata(CustomTestCase):
    """
    All the test cases examples have 1 additional cache location than the decode length.
    This is to align with the current allocation logic. It does not affect the correctness.
    """
```
**EN:** This range declares `TestUpdateDraftDecodeSetExpandMetadata`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 446-464: Test routines around test_draft_decode_set_expand_metadata / 测试例程
```python
    def test_draft_decode_set_expand_metadata(self):
        bs, topk, page_size = 1, 2, 4

        cases = [
            (
                torch.tensor(
                    [
                        [23, 24],
                        [31, 32],
                    ],
                    dtype=torch.int32,
                ),
                torch.tensor(
                    [
                        [5, 6],
                        [7, 8],
                    ],
                    dtype=torch.int32,
                ),
```
**EN:** This range defines concrete test routine(s) `test_draft_decode_set_expand_metadata`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `tensor`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 465-470: Scenario logic / 场景逻辑
```python
                1,
            ),
            # Decode span multiple pages:
            # duplicated kv cache: 24, 25, 26
            # decode locations: 27, 28, 29, 30, 31, 32
            # We need 3 pages in total.
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 471-489: Scenario logic / 场景逻辑
```python
            (
                torch.tensor(
                    [
                        [27, 28, 29, 30, 31, 32],
                        [35, 36, 37, 38, 39, 40],
                    ],
                    dtype=torch.int32,
                ),
                torch.tensor(
                    [
                        [6, 7, 8, 0, 0, 0],
                        [8, 9, 10, 0, 0, 0],
                    ],
                    dtype=torch.int32,
                ),
                5,
            ),
        ]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 490-495: Scenario logic / 场景逻辑
```python
        last_page_lens = torch.tensor([3], dtype=torch.int32)
        for cache_loc, expected_page_table, decode_length in cases:
            cache_seqlens_int32 = torch.zeros(bs * topk, dtype=torch.int32)
            page_table = torch.zeros_like(cache_loc, dtype=torch.int32)
            draft_decode_set_expand_metadata(
                cache_seqlens_int32=cache_seqlens_int32,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`, `zeros`, `zeros_like` and `draft_decode_set_expand_metadata`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 496-508: Assertions and result checks / 断言与结果检查
```python
                page_table=page_table,
                last_page_lens=last_page_lens,
                decode_length=decode_length,
                cache_loc=cache_loc,
                topk=topk,
                page_size=page_size,
            )

            expected_cache_seqlens = torch.tensor(
                [decode_length + 3, decode_length + 3], dtype=torch.int32
            )
            self.assertTrue(torch.equal(cache_seqlens_int32, expected_cache_seqlens))
            self.assertTrue(torch.equal(page_table, expected_page_table))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tensor`, `assertTrue` and `equal`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 510-514: Test routines around test_update_draft_decode_set_expand_metadata_multi_batch / 测试例程
```python
    def test_update_draft_decode_set_expand_metadata_multi_batch(self):
        """
        Ensure expand metadata works when batch size > 1 and last pages differ.
        """
        bs, topk, decode_length, page_size = 3, 2, 3, 4
```
**EN:** This range defines concrete test routine(s) `test_update_draft_decode_set_expand_metadata_multi_batch`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 515-534: Scenario logic / 场景逻辑
```python
        cache_loc = torch.tensor(
            [
                # First batch: last page duplicate is 1, consecutive pages
                [1, 2, 3, 4],
                [6, 7, 8, 9],
                # Second batch: last page duplicate is 3, non-consecutive pages
                [3, 8, 9, 10],
                [14, 15, 16, 17],
                # Third batch: last page duplicate is 0, consecutive pages
                [0, 1, 2, 3],
                [4, 5, 6, 7],
            ],
            dtype=torch.int32,
        )
        cache_seqlens_int32 = torch.zeros(bs * topk, dtype=torch.int32)
        last_page_lens = torch.tensor([1, 3, 0], dtype=torch.int32)
        page_table = torch.zeros_like(cache_loc, dtype=torch.int32)
        draft_decode_set_expand_metadata(
            cache_seqlens_int32=cache_seqlens_int32,
            page_table=page_table,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `tensor`, `zeros`, `zeros_like` and `draft_decode_set_expand_metadata`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 535-539: Scenario logic / 场景逻辑
```python
            last_page_lens=last_page_lens,
            decode_length=decode_length,
            cache_loc=cache_loc,
            topk=topk,
            page_size=page_size,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 540-555: Assertions and result checks / 断言与结果检查
```python
        )

        expected_cache_seqlens = torch.tensor([4, 4, 6, 6, 3, 3], dtype=torch.int32)
        expected_page_table = torch.tensor(
            [
                [0, 1, 0, 0],
                [1, 2, 0, 0],
                [0, 2, 0, 0],
                [3, 4, 0, 0],
                [0, 0, 0, 0],
                [1, 0, 0, 0],
            ],
            dtype=torch.int32,
        )
        self.assertTrue(torch.equal(cache_seqlens_int32, expected_cache_seqlens))
        self.assertTrue(torch.equal(page_table, expected_page_table))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `tensor`, `assertTrue` and `equal`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 556-559: Script entry point / 脚本入口
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
- Multimodal inputs / 多模态输入
- Disaggregated serving / 解耦式服务
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.srt.configs.model_config`, `sglang.srt.layers.attention.flashattention_backend`, `sglang.srt.layers.attention.torch_native_backend`, `sglang.srt.layers.radix_attention`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.model_executor.forward_batch_info`, `sglang.test.test_utils`
