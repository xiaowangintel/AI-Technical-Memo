# test_mha_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_mha_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_mha_attn, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_mha_attn 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-7)
```python
"""
Test:

* Tests for MMEncoderAttention layer
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 9-26)
```python
import itertools
from unittest.mock import patch

import numpy as np
import pytest
import torch

from vllm.config import get_current_vllm_config
from vllm.config.multimodal import MultiModalConfig
from vllm.model_executor.layers.attention import MMEncoderAttention
from vllm.platforms import current_platform
from vllm.platforms.cpu import CpuPlatform
from vllm.platforms.cuda import CudaPlatform
from vllm.platforms.interface import DeviceCapability
from vllm.platforms.rocm import RocmPlatform
from vllm.utils.torch_utils import set_default_torch_dtype, set_random_seed
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.attention.selector import _cached_get_attn_backend
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as itertools, unittest.mock, numpy, pytest; and vLLM components like vllm.config, vllm.config.multimodal, vllm.model_executor.layers.attention, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 itertools、unittest.mock、numpy、pytest；vLLM 内部组件，例如 vllm.config、vllm.config.multimodal、vllm.model_executor.layers.attention、vllm.platforms。

### Function `clear_cache` (lines 29-32)
```python
@pytest.fixture(autouse=True)
def clear_cache():
    """Clear lru cache to ensure each test case runs without caching."""
    _cached_get_attn_backend.cache_clear()
```
**EN:** This fixture prepares reusable state for clear cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 clear cache 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 35-35)
```python
devices = ["cpu"]
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Top-level block starting at line 36 (lines 36-37)
```python
if current_platform.is_cuda():
    devices.append("cuda")
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 38 (lines 38-39)
```python
if current_platform.is_rocm():
    devices.append("hip")
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Function `test_mha_attn_platform` (lines 42-99)
```python
@pytest.mark.parametrize("device", devices)
def test_mha_attn_platform(default_vllm_config, device: str):
    """
    Test the attention selector between different platform and device.
    """
    torch.set_default_dtype(torch.float16)

    if device == "cpu":
        with (
            patch("vllm.model_executor.models.vision.current_platform", CpuPlatform()),
        ):
            attn = MMEncoderAttention(16, 64, scale=1)
            assert attn.attn_backend == AttentionBackendEnum.TORCH_SDPA
    elif device == "hip":
        with (
            patch("vllm.model_executor.models.vision.current_platform", RocmPlatform()),
        ):
            attn = MMEncoderAttention(16, 64, scale=1)
            assert attn.attn_backend == AttentionBackendEnum.FLASH_ATTN
    else:
        # Test CUDA with head_size=64 (divisible by 32)
        # - should use vLLM's FlashAttention
        with (
            patch("vllm.model_executor.models.vision.current_platform", CudaPlatform()),
        ):
            attn = MMEncoderAttention(16, 64, scale=1)
            assert attn.attn_backend == AttentionBackendEnum.FLASH_ATTN

        # Test CUDA with head_size=72 (not divisible by 32)
        # - should use vLLM's FlashAttention
        with (
            patch("vllm.model_executor.models.vision.current_platform", CudaPlatform()),
        ):
            attn = MMEncoderAttention(16, 72, scale=1)
            assert attn.attn_backend == AttentionBackendEnum.FLASH_ATTN

        # Test CUDA with head_size=72 (not divisible by 32)
        # - should use vLLM's FlashAttention
        with (
            patch("vllm.model_executor.models.vision.current_platform", CudaPlatform()),
            set_default_torch_dtype(torch.float32),
        ):
            attn = MMEncoderAttention(16, 72, scale=1)
            assert attn.attn_backend == AttentionBackendEnum.TRITON_ATTN

        # Test Turing (pre-Ampere, sm_75): FlashAttention requires sm>=80,
        # and Triton no longer supports MMA on Turing, so we expect that
        # TORCH_SDPA is used for MMEncoderAttention.
        with (
            patch("vllm.model_executor.models.vision.current_platform", CudaPlatform()),
            patch.object(
                CudaPlatform,
                "get_device_capability",
                return_value=DeviceCapability(major=7, minor=5),
            ),
        ):
            attn = MMEncoderAttention(16, 64, scale=1)
            assert attn.attn_backend == AttentionBackendEnum.TORCH_SDPA
```
**EN:** This pytest case verifies mha attn platform. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, device. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 mha attn platform 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、device 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `ref_attention` (lines 102-117)
```python
def ref_attention(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    scale: float,
) -> torch.Tensor:
    """
    Native implementation of scaled dot product attention without mask:
    - query, key, value: [batch_size, seq_len, num_heads, head_size]
    - attn_mask: [batch_size, seq_len, seq_len]
    """
    query, key, value = (x.transpose(1, 2) for x in (query, key, value))
    attn_weights = scale * torch.matmul(query, key.transpose(2, 3))
    attn_weights = torch.softmax(attn_weights, dim=-1).to(value.dtype)
    out = torch.matmul(attn_weights, value).transpose(1, 2)
    return out
```
**EN:** This helper acts as a reference implementation for attention. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 attention 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 120-135)
```python
BATCH_SIZES = [1, 16]
SEQ_LENS = [1]
VAR_SEQ_LENS = [
    [2, 2],
    [2, 3, 4],
]
NUM_HEADS = [1, 16]
NUM_KV_HEADS = [1]
HEAD_SIZES = [64, 80]
# flshattF and tritonflashattF supported: {torch.float16, torch.bfloat16}
DTYPES = (
    [torch.half, torch.bfloat16, torch.float]
    if not current_platform.is_rocm()
    else [torch.half, torch.bfloat16]
)
CUDA_DEVICES = ["cuda"]
```
**EN:** This block centralizes shared constants and parameter grids, including BATCH_SIZES, SEQ_LENS, VAR_SEQ_LENS, NUM_HEADS, NUM_KV_HEADS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 BATCH_SIZES、SEQ_LENS、VAR_SEQ_LENS、NUM_HEADS、NUM_KV_HEADS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_mha_attn_forward` (lines 138-188)
```python
@pytest.mark.parametrize("batch_size", BATCH_SIZES)
@pytest.mark.parametrize("seq_len", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("num_kv_heads", NUM_KV_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("device", CUDA_DEVICES)
def test_mha_attn_forward(
    default_vllm_config,
    batch_size: int,
    seq_len: int,
    num_heads: int,
    num_kv_heads: int,
    head_size: int,
    dtype: torch.dtype,
    device: str,
):
    set_random_seed(0)
    torch.set_default_device(device)
    torch.set_default_dtype(dtype)

    q = torch.randn(batch_size, seq_len, num_heads * head_size)
    k = torch.randn(batch_size, seq_len, num_kv_heads * head_size)
    v = torch.randn(batch_size, seq_len, num_kv_heads * head_size)
    scale = 1.0 / head_size**0.5
    attn = MMEncoderAttention(
        num_heads, head_size, scale=scale, num_kv_heads=num_kv_heads
    )
    output = attn(q, k, v)

    assert num_heads % num_kv_heads == 0
    num_queries_per_kv = num_heads // num_kv_heads
    q = q.reshape(batch_size, seq_len, num_heads, head_size)
    k = k.reshape(batch_size, seq_len, num_kv_heads, head_size)
    v = v.reshape(batch_size, seq_len, num_kv_heads, head_size)
    if num_queries_per_kv > 1:
        k = torch.repeat_interleave(k, num_queries_per_kv, dim=2)
        v = torch.repeat_interleave(v, num_queries_per_kv, dim=2)

    ref_output = ref_attention(
        q,
        k,
        v,
        scale=scale,
    ).reshape(batch_size, seq_len, num_heads * head_size)
    tol_kwargs = (
        dict(rtol=1e-3, atol=1e-3)
        if attn.attn_backend == AttentionBackendEnum.TRITON_ATTN
        else {}
    )
    torch.testing.assert_close(output, ref_output, **tol_kwargs)
```
**EN:** This pytest case verifies mha attn forward. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, batch_size, seq_len, num_heads. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 mha attn forward 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、batch_size、seq_len、num_heads 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_mha_attn_varlen_forward` (lines 191-244)
```python
@pytest.mark.parametrize("var_seq_len", VAR_SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("num_kv_heads", NUM_KV_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("device", CUDA_DEVICES)
def test_mha_attn_varlen_forward(
    default_vllm_config,
    var_seq_len: list[int],
    num_heads: int,
    num_kv_heads: int,
    head_size: int,
    dtype: torch.dtype,
    device: str,
):
    set_random_seed(0)
    torch.set_default_device(device)
    torch.set_default_dtype(dtype)

    q = torch.randn(1, sum(var_seq_len), num_heads, head_size)
    k = torch.randn(1, sum(var_seq_len), num_kv_heads, head_size)
    v = torch.randn(1, sum(var_seq_len), num_kv_heads, head_size)
    cu_seqlens = torch.tensor(
        [0] + list(itertools.accumulate(var_seq_len)), dtype=torch.int32
    )
    scale = 1.0 / head_size**0.5
    attn = MMEncoderAttention(
        num_heads, head_size, scale=scale, num_kv_heads=num_kv_heads
    )
    output = attn(
        q, k, v, cu_seqlens=cu_seqlens, max_seqlen=torch.tensor(max(var_seq_len))
    )

    assert num_heads % num_kv_heads == 0
    num_queries_per_kv = num_heads // num_kv_heads
    if num_queries_per_kv > 1:
        k = torch.repeat_interleave(k, num_queries_per_kv, dim=2)
        v = torch.repeat_interleave(v, num_queries_per_kv, dim=2)

    ref_output = []
    for q_i, k_i, v_i in zip(
        torch.split(q, var_seq_len, dim=1),
        torch.split(k, var_seq_len, dim=1),
        torch.split(v, var_seq_len, dim=1),
    ):
        output_i = ref_attention(
            q_i,
            k_i,
            v_i,
            scale=scale,
        )
        ref_output.append(output_i)
    ref_output = torch.cat(ref_output, dim=1)
    torch.testing.assert_close(output, ref_output, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies mha attn varlen forward. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, var_seq_len, num_heads, num_kv_heads. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 mha attn varlen forward 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、var_seq_len、num_heads、num_kv_heads 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_mha_attn_varlen_forward_flashinfer` (lines 247-348)
```python
@pytest.mark.parametrize("var_seq_len", VAR_SEQ_LENS)
@pytest.mark.parametrize(
    "dtype",
    [torch.bfloat16, torch.half],
)
@pytest.mark.parametrize("device", CUDA_DEVICES)
def test_mha_attn_varlen_forward_flashinfer(
    default_vllm_config,
    var_seq_len: list[int],
    dtype: torch.dtype,
    device: str,
):
    """Test MMEncoderAttention varlen forward with FLASHINFER backend (head_size=72).

    Exercises the path that uses --mm-encoder-attn-backend=FLASHINFER with
    recomputed cu_seqlens, max_seqlen, and sequence_lengths as in qwen3_vl
    vision encoder.
    """
    pytest.importorskip("flashinfer")

    num_heads = 16
    head_size = 72
    set_random_seed(0)
    torch.set_default_device(device)
    torch.set_default_dtype(dtype)

    # Override vllm config so get_vit_attn_backend returns FLASHINFER (simulates
    # --mm-encoder-attn-backend=FLASHINFER).
    vllm_config = get_current_vllm_config()
    old_model_config = getattr(vllm_config, "model_config", None)
    minimal_model_config = type(
        "MinimalModelConfig",
        (),
        {
            "multimodal_config": MultiModalConfig(
                mm_encoder_attn_backend=AttentionBackendEnum.FLASHINFER
            ),
        },
    )()
    vllm_config.model_config = minimal_model_config
# ... excerpt ...

        cu_seqlens = MMEncoderAttention.maybe_recompute_cu_seqlens(
            AttentionBackendEnum.FLASHINFER,
            cu_seqlens_np,
            hidden_size,
            tp_size,
            device,
        )

        scale = 1.0 / head_size**0.5
        attn = MMEncoderAttention(
            num_heads,
            head_size,
            scale=scale,
            num_kv_heads=num_heads,
        )
        assert attn.attn_backend == AttentionBackendEnum.FLASHINFER

        output = attn(
            q,
            k,
            v,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
            sequence_lengths=sequence_lengths,
        )

        ref_output = []
        for q_i, k_i, v_i in zip(
            torch.split(q, var_seq_len, dim=1),
            torch.split(k, var_seq_len, dim=1),
            torch.split(v, var_seq_len, dim=1),
        ):
            output_i = ref_attention(q_i, k_i, v_i, scale=scale)
            ref_output.append(output_i)
        ref_output = torch.cat(ref_output, dim=1)
        torch.testing.assert_close(output, ref_output, atol=1e-2, rtol=1e-2)
    finally:
        vllm_config.model_config = old_model_config
```
**EN:** This pytest case verifies mha attn varlen forward flashinfer. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, var_seq_len, dtype, device. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 mha attn varlen forward flashinfer 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、var_seq_len、dtype、device 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `itertools`
- `unittest.mock -> patch`
- `numpy`
- `pytest`
- `torch`
- `vllm.config -> get_current_vllm_config`
- `vllm.config.multimodal -> MultiModalConfig`
- `vllm.model_executor.layers.attention -> MMEncoderAttention`
- `vllm.platforms -> current_platform`
- `vllm.platforms.cpu -> CpuPlatform`
- `vllm.platforms.cuda -> CudaPlatform`
- `vllm.platforms.interface -> DeviceCapability`
- `vllm.platforms.rocm -> RocmPlatform`
- `vllm.utils.torch_utils -> set_default_torch_dtype, set_random_seed`
- `vllm.v1.attention.backends.registry -> AttentionBackendEnum`
- `vllm.v1.attention.selector -> _cached_get_attn_backend`
