# test_per_token_kv_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/quantization/test_per_token_kv_cache.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for per-token-head KV cache quantization (INT8 and FP8). / 该文件主要围绕 Per Token KV Cache 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-49)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Tests for per-token-head KV cache quantization (INT8 and FP8).

Covers:
- Per-token-head Triton reshape-and-cache kernel
- Round-trip quantize/dequantize accuracy
- process_weights_after_loading early-return path
- End-to-end integration with Triton unified attention kernel

Run: pytest tests/quantization/test_per_token_kv_cache.py -v -s
"""

import random
from dataclasses import dataclass
from unittest.mock import MagicMock

import pytest
import torch
# ... omitted for brevity ...
]

# ---------------------------------------------------------------------------
# Test parameters
# ---------------------------------------------------------------------------
NUM_TOKENS = [1, 7, 42]
NUM_KV_HEADS = [1, 4, 8]
HEAD_SIZES = [64, 128]
BLOCK_SIZES = [16]
SEEDS = [0]

# Platform-dependent FP8 dtype and range
FP8_DTYPE = current_platform.fp8_dtype()
FP8_MIN, FP8_MAX = get_fp8_min_max()
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `random`, `dataclasses`, `pytest`, `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: QuantConfig (lines 55-65)
```python
@dataclass(frozen=True)
class QuantConfig:
    """Quantization parameters for a given cache dtype."""

    cache_dtype: torch.dtype  # torch.int8 or FP8_DTYPE
    kv_cache_dtype_str: str  # "int8_per_token_head" or "fp8_per_token_head"
    quant_max: float
    quant_min: float
    kv_quant_mode: KVQuantMode
    # INT8 Triton stores truncate; FP8 hardware casts round.
    uses_trunc: bool
```
**EN:** Groups related scenarios for Quantconfig.
**CN:** 该类把与 Quantconfig 相关的场景组织在一起。

### Constants / assignments (lines 68-75)
```python
INT8_CONFIG = QuantConfig(
    cache_dtype=torch.int8,
    kv_cache_dtype_str="int8_per_token_head",
    quant_max=127.0,
    quant_min=-128.0,
    kv_quant_mode=KVQuantMode.INT8_PER_TOKEN_HEAD,
    uses_trunc=True,
)
```
**EN:** Defines shared constants or configuration objects like `INT8_CONFIG`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `INT8_CONFIG`），供后续测试重复使用。

### Fixture: qcfg (lines 88-90)
```python
@pytest.fixture(params=QUANT_CONFIGS, ids=["int8", "fp8"])
def qcfg(request) -> QuantConfig:
    return request.param
```
**EN:** Provides a pytest fixture for Qcfg. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`.
**CN:** 该代码块定义 pytest 夹具 `qcfg`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture` 构造或返回测试所需的值。

### Helper: _quantize_per_token_head_ref (lines 96-111)
```python
def _quantize_per_token_head_ref(
    data: torch.Tensor,  # [num_tokens, num_heads, head_size]
    cfg: QuantConfig,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Reference per-token-head quantization (one scale per token per head).

    Returns (quantized, scales) where scales is [num_tokens, num_heads].
    """
    absmax = data.float().abs().amax(dim=2)  # [num_tokens, num_heads]
    scales = (absmax / cfg.quant_max).clamp(min=1e-6)
    scaled = data.float() * (1.0 / scales[:, :, None])
    if cfg.uses_trunc:
        q = scaled.round().clamp(cfg.quant_min, cfg.quant_max).to(cfg.cache_dtype)
    else:
        q = scaled.clamp(cfg.quant_min, cfg.quant_max).to(cfg.cache_dtype)
    return q, scales
```
**EN:** Reference per-token-head quantization (one scale per token per head). It coordinates operations such as `data.float().abs().amax`, `(absmax / cfg.quant_max).clamp`, `data.float`.
**CN:** 该辅助函数为 Quantize Per Token Head Ref 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `data.float().abs().amax`, `(absmax / cfg.quant_max).clamp`, `data.float` 等操作。

### Class: TestIsQuantizedKvCache (lines 117-145)
```python
class TestIsQuantizedKvCache:
    def test_fp8_variants(self):
        assert is_quantized_kv_cache("fp8")
        assert is_quantized_kv_cache("fp8_e4m3")
        assert is_quantized_kv_cache("fp8_e5m2")

    def test_int8_per_token_head(self):
        assert is_quantized_kv_cache("int8_per_token_head")

    def test_fp8_per_token_head(self):
        assert is_quantized_kv_cache("fp8_per_token_head")

    def test_auto(self):
        assert not is_quantized_kv_cache("auto")

    def test_bfloat16(self):
        assert not is_quantized_kv_cache("bfloat16")

    def test_kv_quant_mode_int8(self):
        from vllm.v1.kv_cache_interface import get_kv_quant_mode

        assert (
            get_kv_quant_mode("int8_per_token_head") == KVQuantMode.INT8_PER_TOKEN_HEAD
        )

    def test_kv_quant_mode_fp8(self):
        from vllm.v1.kv_cache_interface import get_kv_quant_mode

        assert get_kv_quant_mode("fp8_per_token_head") == KVQuantMode.FP8_PER_TOKEN_HEAD
```
**EN:** Groups related scenarios for Testisquantizedkvcache. The class contains 7 test method(s).
**CN:** 该类把与 Testisquantizedkvcache 相关的场景组织在一起。 其中包含 7 个测试方法。

### Test: test_reshape_and_cache_per_token_head (lines 151-238)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("num_heads", NUM_KV_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("seed", SEEDS)
@torch.inference_mode()
def test_reshape_and_cache_per_token_head(
    qcfg: QuantConfig,
    num_tokens: int,
    num_heads: int,
    head_size: int,
    block_size: int,
    seed: int,
):
    """Test triton_reshape_and_cache_flash_per_token_head_quant kernel."""
    from vllm.v1.attention.ops.triton_reshape_and_cache_flash import (
        triton_reshape_and_cache_flash_per_token_head_quant,
    )

# ... omitted for brevity ...
        v_ref_deq = value[i].float()
        torch.testing.assert_close(
            v_deq,
            v_ref_deq,
            atol=0.1,
            rtol=0.1,
        )
        # Per-head scales: [num_heads]
        torch.testing.assert_close(
            k_scale_cache[blk, off], ref_k_scales[i], atol=1e-4, rtol=1e-3
        )
        torch.testing.assert_close(
            v_scale_cache[blk, off], ref_v_scales[i], atol=1e-4, rtol=1e-3
        )
```
**EN:** Test triton_reshape_and_cache_flash_per_token_head_quant kernel. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `torch.inference_mode`, `set_random_seed` before asserting the expected outcome.
**CN:** 该测试用例验证 Reshape And Cache Per Token Head 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `torch.inference_mode`, `set_random_seed` 驱动目标逻辑，再断言预期结果。

### Test: test_per_token_head_round_trip_accuracy (lines 244-315)
```python
@pytest.mark.parametrize("num_tokens", [1, 16])
@pytest.mark.parametrize("num_heads", [4])
@pytest.mark.parametrize("head_size", [128])
@pytest.mark.parametrize("block_size", [16])
@torch.inference_mode()
def test_per_token_head_round_trip_accuracy(
    qcfg: QuantConfig,
    num_tokens: int,
    num_heads: int,
    head_size: int,
    block_size: int,
):
    """Verify per-token-head round-trip: kernel dequant matches reference.

    INT8: Triton truncates on float->int8 store.
    FP8: hardware cast (clamp then cast).
    """
    from vllm.v1.attention.ops.triton_reshape_and_cache_flash import (
        triton_reshape_and_cache_flash_per_token_head_quant,
# ... omitted for brevity ...
            for h in range(num_heads):
                orig = data[i, h].float()  # [head_size]

                actual_q = cache[blk, off, h]
                actual_sc = sc[blk, off, h]
                actual_deq = actual_q.float() * actual_sc

                # Round-trip: dequantized should be close to original
                torch.testing.assert_close(
                    actual_deq,
                    orig,
                    atol=0.1,
                    rtol=0.1,
                )
```
**EN:** Verify per-token-head round-trip: kernel dequant matches reference. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `torch.inference_mode`, `torch.set_default_device` before asserting the expected outcome.
**CN:** 该测试用例验证 Per Token Head Round Trip Accuracy 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `torch.inference_mode`, `torch.set_default_device` 驱动目标逻辑，再断言预期结果。

### Test: test_per_token_head_negative_slot_skipped (lines 321-370)
```python
@torch.inference_mode()
def test_per_token_head_negative_slot_skipped(qcfg: QuantConfig):
    """Tokens with slot_mapping=-1 should leave the cache unchanged."""
    from vllm.v1.attention.ops.triton_reshape_and_cache_flash import (
        triton_reshape_and_cache_flash_per_token_head_quant,
    )

    torch.set_default_device(DEVICE_TYPE)
    num_tokens = 4
    num_heads = 2
    head_size = 64
    block_size = 16
    num_blocks = 2

    key = torch.randn(num_tokens, num_heads, head_size, dtype=torch.bfloat16)
    value = torch.randn(num_tokens, num_heads, head_size, dtype=torch.bfloat16)

    key_cache = torch.zeros(
        num_blocks, block_size, num_heads, head_size, dtype=qcfg.cache_dtype
# ... omitted for brevity ...
        k_scale_cache,
        v_scale_cache,
        slot_mapping,
    )

    # Slots 0 and 1 should have been written (tokens 0 and 2)
    assert not torch.equal(key_cache[0, 0], key_cache_before[0, 0])
    assert not torch.equal(key_cache[0, 1], key_cache_before[0, 1])
    assert not torch.equal(value_cache[0, 0], val_cache_before[0, 0])

    # All other slots should be unchanged
    assert torch.equal(key_cache[0, 2:], key_cache_before[0, 2:])
    assert torch.equal(key_cache[1], key_cache_before[1])
    assert torch.equal(value_cache[0, 2:], val_cache_before[0, 2:])
```
**EN:** Tokens with slot_mapping=-1 should leave the cache unchanged. The body exercises logic via `torch.inference_mode`, `torch.set_default_device`, `torch.randn` before asserting the expected outcome.
**CN:** 该测试用例验证 Per Token Head Negative Slot Skipped 在特定场景下的行为。 函数体会先通过 `torch.inference_mode`, `torch.set_default_device`, `torch.randn` 驱动目标逻辑，再断言预期结果。

### Test: test_process_weights_sets_placeholder_scales (lines 376-407)
```python
@pytest.mark.parametrize(
    "kv_cache_dtype", ["int8_per_token_head", "fp8_per_token_head"]
)
def test_process_weights_sets_placeholder_scales(kv_cache_dtype: str):
    """Per-token-head should set _k_scale=1.0, _v_scale=1.0
    and delete checkpoint attrs."""
    from vllm.model_executor.layers.quantization.kv_cache import (
        BaseKVCacheMethod,
    )

    layer = MagicMock()
    layer.kv_cache_dtype = kv_cache_dtype
    layer.calculate_kv_scales = False
    layer.k_scale = torch.nn.Parameter(torch.tensor(-1.0), requires_grad=False)
    layer.v_scale = torch.nn.Parameter(torch.tensor(-1.0), requires_grad=False)
    layer.q_scale = torch.nn.Parameter(torch.tensor(-1.0), requires_grad=False)
    layer.prob_scale = torch.nn.Parameter(torch.tensor(-1.0), requires_grad=False)
    layer._k_scale = torch.tensor(0.0)
    layer._v_scale = torch.tensor(0.0)
    layer._k_scale_float = 0.0
    layer._v_scale_float = 0.0

    method = BaseKVCacheMethod.__new__(BaseKVCacheMethod)
    method.quant_config = MagicMock()
    method.process_weights_after_loading(layer)

    assert layer._k_scale_float == 1.0
    assert layer._v_scale_float == 1.0
    assert not hasattr(layer, "k_scale")
    assert not hasattr(layer, "v_scale")
    assert not hasattr(layer, "q_scale")
    assert not hasattr(layer, "prob_scale")
```
**EN:** Per-token-head should set _k_scale=1.0, _v_scale=1.0 and delete checkpoint attrs. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `MagicMock`, `torch.nn.Parameter` before asserting the expected outcome.
**CN:** 该测试用例验证 Process Weights Sets Placeholder Scales 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `MagicMock`, `torch.nn.Parameter` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
Constants / assignments
Constants / assignments
test_triton_unified_attention_per_token_head_scale
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Quantization / 量化**
  - **EN:** The file covers low-precision model paths, kernels, or configuration rules for quantized inference.
  - **CN:** 该文件覆盖低精度模型路径、相关内核或量化推理配置规则。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `dataclasses`, `unittest.mock`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.torch_utils`, `vllm.v1.kv_cache_interface`, `vllm.v1.attention.ops.triton_reshape_and_cache_flash`, `vllm.model_executor.layers.quantization.kv_cache`, `vllm.utils.math_utils`, `vllm.v1.attention.ops.triton_unified_attention`
