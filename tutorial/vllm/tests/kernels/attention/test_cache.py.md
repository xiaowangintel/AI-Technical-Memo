# test_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_cache.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_cache, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_cache 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-13)
```python
import random

import pytest
import torch

from tests.kernels.utils import DEFAULT_OPCHECK_TEST_UTILS, opcheck
from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.quant_utils import scaled_dequantize
from vllm.platforms import current_platform
from vllm.utils.torch_utils import nvfp4_kv_cache_split_views, set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as random, pytest, torch; shared test helpers from tests.kernels.utils; and vLLM components like vllm, vllm.model_executor.layers.quantization.utils.quant_utils, vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 random、pytest、torch；共享测试辅助模块，例如 tests.kernels.utils；vLLM 内部组件，例如 vllm、vllm.model_executor.layers.quantization.utils.quant_utils、vllm.platforms、vllm.utils.torch_utils。

### Constants and module state (lines 15-45)
```python
COPYING_DIRECTION = [("cuda", "cpu"), ("cuda", "cuda"), ("cpu", "cuda")]
DTYPES = [torch.bfloat16, torch.float]
NUM_TOKENS = [42]  # Arbitrary values for testing
NUM_LAYERS = [1]  # Arbitrary values for testing
NUM_HEADS = [8]  # Arbitrary values for testing
HEAD_SIZES = [64, 80, 256]
BLOCK_SIZES = [8, 16, 32]
CACHE_LAYOUTS = ["NHD", "HND"]
KV_SCALE_TYPES = ["tensor", "attn_head"]

# Parameters for MLA tests.
KV_LORA_RANKS = [256, 512]
QK_ROPE_HEAD_DIMS = [64]
NUM_TOKENS_MLA = [42]
BLOCK_SIZES_MLA = [16]
NUM_BLOCKS_MLA = [8]

# Arbitrary values for testing
# don't make it too large. e.g. [1024, 36000] will OOM
NUM_BLOCKS = [1024, 10000]

NUM_MAPPINGS = [256]  # Arbitrary values for testing
SEEDS = [0]
CUDA_DEVICES = [
    f"cuda:{i}" for i in range(1 if torch.accelerator.device_count() == 1 else 2)
]

# We assume fp8 is always enabled for testing.
KV_CACHE_DTYPE = ["auto", "fp8"]

RESHAPE_FLASH_IMPLEMENTATIONS = ["cuda", "triton"]
```
**EN:** This block centralizes shared constants and parameter grids, including COPYING_DIRECTION, DTYPES, NUM_TOKENS, NUM_LAYERS, NUM_HEADS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 COPYING_DIRECTION、DTYPES、NUM_TOKENS、NUM_LAYERS、NUM_HEADS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_reshape_and_cache` (lines 48-164)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPE)
@torch.inference_mode()
def test_reshape_and_cache(
    kv_cache_factory,
    num_tokens: int,
    num_heads: int,
    head_size: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    seed: int,
    device: str,
    kv_cache_dtype: str,
) -> None:
    if kv_cache_dtype == "fp8" and head_size % 16:
        pytest.skip()
    set_random_seed(seed)
    torch.set_default_device(device)
    torch.accelerator.set_device_index(device)
    # Create a random slot mapping.
    num_slots = block_size * num_blocks
    slot_mapping_lst = random.sample(range(num_slots), num_tokens)
    slot_mapping = torch.tensor(slot_mapping_lst, dtype=torch.long)

    qkv = torch.randn(num_tokens, 3, num_heads, head_size, dtype=dtype)
    _, key, value = qkv.unbind(dim=1)

    # Create the KV caches.
    key_caches, value_caches = kv_cache_factory(
        num_blocks,
        block_size,
        1,
# ... excerpt ...
    ops.reshape_and_cache(
        key,
        value,
        key_cache,
        value_cache,
        slot_mapping,
        kv_cache_dtype,
        k_scale,
        v_scale,
    )

    if kv_cache_dtype == "fp8":
        result_key_cache = torch.empty_like(key_cache, dtype=torch.float16)
        ops.convert_fp8(result_key_cache, key_cache, k_scale.item())
        result_value_cache = torch.empty_like(value_cache, dtype=torch.float16)
        ops.convert_fp8(result_value_cache, value_cache, v_scale.item())

    # Run the reference implementation.
    reshaped_key = key.reshape(num_tokens, *key_cache[0, :, :, 0, :].shape)
    block_indices = torch.div(slot_mapping, block_size, rounding_mode="floor")
    block_indices_lst = block_indices.cpu().tolist()
    block_offsets = slot_mapping % block_size
    block_offsets_lst = block_offsets.cpu().tolist()
    for i in range(num_tokens):
        block_idx = block_indices_lst[i]
        block_offset = block_offsets_lst[i]
        cloned_key_cache[block_idx, :, :, block_offset, :] = reshaped_key[i]
        cloned_value_cache[block_idx, :, :, block_offset] = value[i]

    if kv_cache_dtype == "fp8":
        torch.testing.assert_close(
            result_key_cache, cloned_key_cache, atol=0.001, rtol=0.1
        )
        torch.testing.assert_close(
            result_value_cache, cloned_value_cache, atol=0.001, rtol=0.1
        )
    else:
        torch.testing.assert_close(key_cache, cloned_key_cache)
        torch.testing.assert_close(value_cache, cloned_value_cache)
```
**EN:** This pytest case verifies reshape and cache. It is parameterized across 9 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_cache_factory, num_tokens, num_heads, head_size. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 reshape and cache 的行为。 它通过 9 组参数化输入覆盖多种场景；它会使用诸如 kv_cache_factory、num_tokens、num_heads、head_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。 由于该代码块较大，这里只展示关键片段。

### Function `test_reshape_and_cache_flash` (lines 167-428)
```python
@pytest.mark.parametrize("num_tokens", NUM_TOKENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPE + ["nvfp4"])
@pytest.mark.parametrize("kv_cache_layout", CACHE_LAYOUTS)
@pytest.mark.parametrize("kv_scale_type", KV_SCALE_TYPES)
@pytest.mark.parametrize("implementation", RESHAPE_FLASH_IMPLEMENTATIONS)
@torch.inference_mode()
def test_reshape_and_cache_flash(
    kv_cache_factory_flashinfer,
    num_tokens: int,
    num_heads: int,
    head_size: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    seed: int,
    device: str,
    kv_cache_dtype: str,
    kv_cache_layout: str,
    kv_scale_type: str,
    implementation: str,
) -> None:
    set_random_seed(seed)
    torch.set_default_device(device)
    torch.accelerator.set_device_index(device)
    assert implementation in ["cuda", "triton"]
    if implementation == "triton" and kv_cache_layout == "HND":
        pytest.skip("Triton implementation only supports NHD layout.")

    if kv_scale_type == "attn_head" and implementation != "cuda":
        pytest.skip("Only CUDA implementation supports attn_head scaling.")

    if kv_cache_dtype == "nvfp4":
        if not current_platform.has_device_capability(100):
# ... excerpt ...
    key_cache_compact = permute_and_compact(key_cache)
    value_cache_compact = permute_and_compact(value_cache)

    if kv_cache_dtype == "fp8":
        result_key_cache = torch.empty_like(key_cache_compact, dtype=torch.float16)
        convert_fp8_local(result_key_cache, key_cache_compact, k_scale, kv_cache_dtype)
        result_value_cache = torch.empty_like(value_cache_compact, dtype=torch.float16)
        convert_fp8_local(
            result_value_cache,
            value_cache_compact,
            v_scale,
            kv_cache_dtype,
        )

    # Run the reference implementation.
    block_indices = torch.div(slot_mapping, block_size, rounding_mode="floor")
    block_indices_lst = block_indices.cpu().tolist()
    block_offsets = slot_mapping % block_size
    block_offsets_lst = block_offsets.cpu().tolist()
    for i in range(num_tokens):
        block_idx = block_indices_lst[i]
        block_offset = block_offsets_lst[i]
        if kv_cache_layout == "NHD":
            cloned_key_cache[block_idx, block_offset, :, :] = key[i]
            cloned_value_cache[block_idx, block_offset, :, :] = value[i]
        else:
            cloned_key_cache[block_idx, :, block_offset, :] = key[i]
            cloned_value_cache[block_idx, :, block_offset, :] = value[i]

    if kv_cache_dtype == "fp8":
        torch.testing.assert_close(
            result_key_cache, cloned_key_cache, atol=0.001, rtol=0.1
        )
        torch.testing.assert_close(
            result_value_cache, cloned_value_cache, atol=0.001, rtol=0.1
        )
    else:
        torch.testing.assert_close(key_cache_compact, cloned_key_cache)
        torch.testing.assert_close(value_cache_compact, cloned_value_cache)
```
**EN:** This pytest case verifies reshape and cache flash. It is parameterized across 12 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_cache_factory_flashinfer, num_tokens, num_heads, head_size. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 reshape and cache flash 的行为。 它通过 12 组参数化输入覆盖多种场景；它会使用诸如 kv_cache_factory_flashinfer、num_tokens、num_heads、head_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。 由于该代码块较大，这里只展示关键片段。

### Function `test_swap_blocks` (lines 431-551)
```python
@pytest.mark.parametrize("direction", COPYING_DIRECTION)
@pytest.mark.parametrize("num_mappings", NUM_MAPPINGS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPE)
@torch.inference_mode()
def test_swap_blocks(
    kv_cache_factory,
    direction: tuple[str, str],
    num_mappings: int,
    num_heads: int,
    head_size: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    seed: int,
    device: str,
    kv_cache_dtype: str,
) -> None:
    if kv_cache_dtype == "fp8" and "cpu" in direction:
        pytest.skip()
    if kv_cache_dtype == "fp8" and head_size % 16:
        pytest.skip()

    set_random_seed(seed)

    src_device = device if direction[0] == "cuda" else "cpu"
    dst_device = device if direction[1] == "cuda" else "cpu"

    src_blocks = random.sample(range(num_blocks), num_mappings)
    # For the same device, mapping must not overlap
    if src_device == dst_device:
        remaining_blocks = list(set(range(num_blocks)) - set(src_blocks))
        dst_blocks = random.sample(remaining_blocks, num_mappings)
    else:
# ... excerpt ...
        (
            src_key_caches[0],
            dist_key_caches[0],
            block_size_in_bytes,
            block_mapping_tensor,
        ),
        cond=do_opcheck,
    )
    opcheck(
        torch.ops._C_cache_ops.swap_blocks,
        (
            src_value_caches[0],
            dist_value_caches[0],
            block_size_in_bytes,
            block_mapping_tensor,
        ),
        cond=do_opcheck,
    )

    ops.swap_blocks(
        src_key_caches[0],
        dist_key_caches[0],
        block_size_in_bytes,
        block_mapping_tensor,
    )
    ops.swap_blocks(
        src_value_caches[0],
        dist_value_caches[0],
        block_size_in_bytes,
        block_mapping_tensor,
    )

    for src, dst in block_mapping:
        torch.testing.assert_close(
            src_key_caches_clone[src].cpu(), dist_key_caches[0][dst].cpu()
        )
        torch.testing.assert_close(
            src_value_caches_clone[src].cpu(), dist_value_caches[0][dst].cpu()
        )
```
**EN:** This pytest case verifies swap blocks. It is parameterized across 10 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_cache_factory, direction, num_mappings, num_heads. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 swap blocks 的行为。 它通过 10 组参数化输入覆盖多种场景；它会使用诸如 kv_cache_factory、direction、num_mappings、num_heads 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。 由于该代码块较大，这里只展示关键片段。

### Function `test_fp8_e4m3_conversion` (lines 554-585)
```python
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@torch.inference_mode()
def test_fp8_e4m3_conversion(
    num_heads: int,
    head_size: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    seed: int,
    device: str,
) -> None:
    set_random_seed(seed)

    low = -224.0
    high = 224.0
    shape = (num_blocks, num_heads, head_size, block_size)
    cache = torch.empty(shape, dtype=dtype, device=device)
    cache.uniform_(low, high)

    cache_fp8 = torch.empty_like(cache, dtype=torch.uint8)
    ops.convert_fp8(cache_fp8, cache)

    converted_cache = torch.empty_like(cache)
    ops.convert_fp8(converted_cache, cache_fp8)

    torch.testing.assert_close(cache, converted_cache, atol=0.001, rtol=0.1)
```
**EN:** This pytest case verifies FP8 e4m3 conversion. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_heads, head_size, block_size, num_blocks. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 FP8 e4m3 conversion 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 num_heads、head_size、block_size、num_blocks 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `_create_mla_cache` (lines 588-599)
```python
def _create_mla_cache(
    num_blocks: int,
    block_size: int,
    entry_size: int,
    dtype: torch.dtype,
    kv_cache_dtype: str,
    device: str,
) -> torch.Tensor:
    cache_dtype = torch.uint8 if kv_cache_dtype == "fp8" else dtype
    return torch.zeros(
        num_blocks, block_size, entry_size, dtype=cache_dtype, device=device
    )
```
**EN:** This helper function implements the shared logic for create mla cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 create mla cache 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_fill_mla_cache` (lines 602-610)
```python
def _fill_mla_cache(cache: torch.Tensor, kv_cache_dtype: str):
    rand_dtype = torch.float16 if kv_cache_dtype == "fp8" else cache.dtype

    vals = torch.randn(*cache.shape, device=cache.device, dtype=rand_dtype)
    if kv_cache_dtype == "fp8":
        temp = torch.zeros_like(cache)
        ops.convert_fp8(temp, vals, 1.0, kv_dtype=kv_cache_dtype)
        vals = temp
    cache.copy_(vals)
```
**EN:** This helper function implements the shared logic for fill mla cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 fill mla cache 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_concat_and_cache_mla` (lines 613-684)
```python
@pytest.mark.parametrize("kv_lora_rank", KV_LORA_RANKS)
@pytest.mark.parametrize("qk_rope_head_dim", QK_ROPE_HEAD_DIMS)
@pytest.mark.parametrize("num_tokens", NUM_TOKENS_MLA)
@pytest.mark.parametrize("block_size", BLOCK_SIZES_MLA)
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS_MLA)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPE)
@torch.inference_mode()
def test_concat_and_cache_mla(
    kv_lora_rank: int,
    qk_rope_head_dim: int,
    num_tokens: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    seed: int,
    device: str,
    kv_cache_dtype: str,
) -> None:
    set_random_seed(seed)
    torch.set_default_device(device)
    torch.accelerator.set_device_index(device)

    total_slots = num_blocks * block_size
    slot_mapping_lst = random.sample(range(total_slots), num_tokens)
    slot_mapping = torch.tensor(slot_mapping_lst, dtype=torch.long, device=device)

    kv_c = torch.randn(num_tokens, kv_lora_rank, dtype=dtype, device=device)
    k_pe = torch.randn(num_tokens, qk_rope_head_dim, dtype=dtype, device=device)
    entry_size = kv_lora_rank + qk_rope_head_dim

    scale = torch.tensor(0.1, dtype=torch.float32, device=device)
    kv_cache = _create_mla_cache(
        num_blocks, block_size, entry_size, dtype, kv_cache_dtype, device
    )
    ref_temp = torch.zeros(*kv_cache.shape, dtype=dtype, device=device)

    for i in range(num_tokens):
        slot = slot_mapping[i].item()
        block_idx = slot // block_size
        block_offset = slot % block_size
        ref_temp[block_idx, block_offset, :kv_lora_rank] = kv_c[i]
        ref_temp[block_idx, block_offset, kv_lora_rank:] = k_pe[i]

    if kv_cache_dtype == "fp8":
        ref_kv_cache = torch.empty_like(ref_temp, dtype=kv_cache.dtype)
        ops.convert_fp8(ref_kv_cache, ref_temp, scale.item(), kv_dtype=kv_cache_dtype)
    else:
        ref_kv_cache = ref_temp

    opcheck(
        torch.ops._C_cache_ops.concat_and_cache_mla,
        (kv_c, k_pe, kv_cache, slot_mapping, kv_cache_dtype, scale),
        test_utils=DEFAULT_OPCHECK_TEST_UTILS,
    )

    ops.concat_and_cache_mla(kv_c, k_pe, kv_cache, slot_mapping, kv_cache_dtype, scale)

    if kv_cache_dtype == "fp8":
        result_temp = torch.empty_like(kv_cache, dtype=torch.float16)
        ops.convert_fp8(
            result_temp, kv_cache.contiguous(), scale.item(), kv_dtype=kv_cache_dtype
        )
        expected_temp = torch.empty_like(ref_kv_cache, dtype=torch.float16)
        ops.convert_fp8(
            expected_temp, ref_kv_cache, scale.item(), kv_dtype=kv_cache_dtype
        )
        torch.testing.assert_close(result_temp, expected_temp, atol=0.001, rtol=0.1)
    else:
        torch.testing.assert_close(kv_cache, ref_kv_cache)
```
**EN:** This pytest case verifies concat and cache mla. It is parameterized across 9 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lora_rank, qk_rope_head_dim, num_tokens, block_size. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 concat and cache mla 的行为。 它通过 9 组参数化输入覆盖多种场景；它会使用诸如 kv_lora_rank、qk_rope_head_dim、num_tokens、block_size 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

### Function `test_concat_and_cache_ds_mla` (lines 687-803)
```python
@pytest.mark.parametrize("kv_lora_rank", KV_LORA_RANKS)
@pytest.mark.parametrize("qk_rope_head_dim", QK_ROPE_HEAD_DIMS)
@pytest.mark.parametrize("num_tokens", NUM_TOKENS_MLA)
@pytest.mark.parametrize("block_size", BLOCK_SIZES_MLA)
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS_MLA)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@torch.inference_mode()
def test_concat_and_cache_ds_mla(
    kv_lora_rank: int,
    qk_rope_head_dim: int,
    num_tokens: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    seed: int,
    device: str,
) -> None:
    if current_platform.is_rocm():
        pytest.skip("concat_and_cache_mla doesn't support fp8_ds_mla on ROCm")
    if dtype.itemsize != 2:
        pytest.skip("ds_mla only supports 16-bit input")
    if kv_lora_rank != 512:
        pytest.skip("fp8_ds_mla requires kv_lora_rank == 512")
    kv_cache_dtype = "fp8_ds_mla"
    set_random_seed(seed)
    torch.set_default_device(device)
    torch.accelerator.set_device_index(device)

    total_slots = num_blocks * block_size
    slot_mapping_lst = random.sample(range(total_slots), num_tokens)
    slot_mapping = torch.tensor(slot_mapping_lst, dtype=torch.long, device=device)

    kv_c = torch.randn(num_tokens, kv_lora_rank, dtype=dtype, device=device)
    k_pe = torch.randn(num_tokens, qk_rope_head_dim, dtype=dtype, device=device)
    entry_size = kv_lora_rank + (4 * 4) + (2 * qk_rope_head_dim)

    scale = torch.tensor(1.0, dtype=torch.float32, device=device)
    kv_cache = _create_mla_cache(
# ... excerpt ...
            ops.convert_fp8(
                ref_cache_slice[tile_start:tile_end],
                tile_data,
                tile_scale.item(),
                kv_dtype="fp8",
            )

        for j in range(qk_rope_head_dim):
            ref_cache_16bit[kv_lora_rank // 2 + 8 + j] = k_pe[i, j]

    opcheck(
        torch.ops._C_cache_ops.concat_and_cache_mla,
        (kv_c, k_pe, kv_cache, slot_mapping, kv_cache_dtype, scale),
        test_utils=DEFAULT_OPCHECK_TEST_UTILS,
    )

    ops.concat_and_cache_mla(kv_c, k_pe, kv_cache, slot_mapping, kv_cache_dtype, scale)

    for i in range(num_tokens):
        slot = slot_mapping[i].item()
        block_idx = slot // block_size
        block_offset = slot % block_size
        kv_cache_slice = kv_cache[block_idx, block_offset]
        ref_cache_slice = ref_cache[block_idx, block_offset]

        kv_nope = kv_cache_slice[:kv_lora_rank]
        ref_nope = ref_cache_slice[:kv_lora_rank]
        kv_scales = kv_cache_slice.view(torch.float32)[
            kv_lora_rank // 4 : kv_lora_rank // 4 + 4
        ]
        ref_scales = ref_cache_slice.view(torch.float32)[
            kv_lora_rank // 4 : kv_lora_rank // 4 + 4
        ]
        kv_rope = kv_cache_slice.view(dtype)[kv_lora_rank // 2 + 8 :]
        ref_rope = ref_cache_slice.view(dtype)[kv_lora_rank // 2 + 8 :]

        torch.testing.assert_close(kv_nope, ref_nope, atol=0.001, rtol=0.1)
        torch.testing.assert_close(kv_scales, ref_scales, atol=0.001, rtol=0.1)
        torch.testing.assert_close(kv_rope, ref_rope, atol=0.001, rtol=0.1)
```
**EN:** This pytest case verifies concat and cache ds mla. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lora_rank, qk_rope_head_dim, num_tokens, block_size. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 concat and cache ds mla 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 kv_lora_rank、qk_rope_head_dim、num_tokens、block_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。 由于该代码块较大，这里只展示关键片段。

### Function `test_swap_blocks_mla` (lines 806-867)
```python
@pytest.mark.parametrize("kv_lora_rank", KV_LORA_RANKS)
@pytest.mark.parametrize("qk_rope_head_dim", QK_ROPE_HEAD_DIMS)
@pytest.mark.parametrize("block_size", BLOCK_SIZES_MLA)
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS_MLA)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPE)
@torch.inference_mode()
def test_swap_blocks_mla(
    kv_lora_rank: int,
    qk_rope_head_dim: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    seed: int,
    device: str,
    kv_cache_dtype: str,
) -> None:
    set_random_seed(seed)
    torch.set_default_device(device)
    torch.accelerator.set_device_index(device)

    entry_size = kv_lora_rank + qk_rope_head_dim

    src_cache = _create_mla_cache(
        num_blocks, block_size, entry_size, dtype, kv_cache_dtype, device
    )
    dst_cache = _create_mla_cache(
        num_blocks, block_size, entry_size, dtype, kv_cache_dtype, device
    )

    _fill_mla_cache(src_cache, kv_cache_dtype)
    _fill_mla_cache(dst_cache, kv_cache_dtype)

    src_cache_clone = src_cache.clone()

    num_mappings = min(2, num_blocks // 2)
    src_blocks = random.sample(range(num_blocks), num_mappings)
    remaining_blocks = list(set(range(num_blocks)) - set(src_blocks))
    dst_blocks = random.sample(remaining_blocks, num_mappings)
    block_mapping = list(zip(src_blocks, dst_blocks))
    block_mapping_tensor = torch.tensor(
        block_mapping, dtype=torch.int64, device="cpu"
    ).view(-1, 2)

    block_size_in_bytes = src_cache.element_size() * src_cache.stride(0)
    opcheck(
        torch.ops._C_cache_ops.swap_blocks,
        (src_cache, dst_cache, block_size_in_bytes, block_mapping_tensor),
        test_utils=DEFAULT_OPCHECK_TEST_UTILS,
    )

    ops.swap_blocks(src_cache, dst_cache, block_size_in_bytes, block_mapping_tensor)

    for src, dst in block_mapping:
        torch.testing.assert_close(
            src_cache_clone[src].cpu(),
            dst_cache[dst].cpu(),
            msg=f"Block {src} from src should have been swapped to block "
            f"{dst} in dst_cache.",
        )
```
**EN:** This pytest case verifies swap blocks mla. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lora_rank, qk_rope_head_dim, block_size, num_blocks. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 swap blocks mla 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 kv_lora_rank、qk_rope_head_dim、block_size、num_blocks 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

### Function `test_gather_and_maybe_dequant_cache_mla` (lines 870-978)
```python
@pytest.mark.parametrize("kv_lora_rank", [512])
@pytest.mark.parametrize("qk_rope_head_dim", [64])
@pytest.mark.parametrize("block_size", [16])
@pytest.mark.parametrize("num_blocks", [1024])
@pytest.mark.parametrize("max_seq_len", [512])
@pytest.mark.parametrize("batch_size", [8])
@pytest.mark.parametrize("dtype", [torch.float32])
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8"])
@pytest.mark.parametrize("device", CUDA_DEVICES)
@torch.inference_mode()
def test_gather_and_maybe_dequant_cache_mla(
    kv_lora_rank,
    qk_rope_head_dim,
    block_size,
    num_blocks,
    max_seq_len,
    batch_size,
    dtype,
    kv_cache_dtype,
    device,
):
    entry_size = kv_lora_rank + qk_rope_head_dim
    scale = torch.tensor(0.1, dtype=torch.float32, device=device)
    src_cache = _create_mla_cache(
        num_blocks, block_size, entry_size, dtype, kv_cache_dtype, device
    )
    _fill_mla_cache(src_cache, kv_cache_dtype=kv_cache_dtype)

    seq_len_tensor = torch.randint(
        max_seq_len, max_seq_len + 1, (batch_size,), device=device
    )

    total_tokens = seq_len_tensor.sum()
    cu_seq_lens = torch.empty((batch_size + 1), dtype=torch.int32, device=device)
    cu_seq_lens[0] = 0
    cu_seq_lens[1:] = seq_len_tensor.cumsum(dim=0).to(dtype=torch.int32)
    token_to_seq = torch.arange(0, batch_size, dtype=torch.int32, device=device)
    token_to_seq = torch.repeat_interleave(token_to_seq, seq_len_tensor)
    print("seq_len_tensor", seq_len_tensor)

# ... excerpt ...
        if kv_cache_dtype == "fp8":
            dequantized_last_block = torch.empty_like(last_block_data, dtype=dtype)
            ops.convert_fp8(dequantized_last_block, last_block_data, scale.item())
            gathered_rows.append(dequantized_last_block)
        else:
            gathered_rows.append(last_block_data)

        batch_expected = torch.cat(gathered_rows, dim=0)
        expected_batches.append(batch_expected)
    expected = torch.cat(expected_batches, dim=0)

    opcheck(
        torch.ops._C_cache_ops.gather_and_maybe_dequant_cache,
        (
            src_cache,
            dst,
            block_table,
            cu_seq_lens,
            token_to_seq,
            total_tokens,
            kv_cache_dtype,
            scale,
            None,
        ),
        test_utils=DEFAULT_OPCHECK_TEST_UTILS,
    )

    ops.gather_and_maybe_dequant_cache(
        src_cache,
        dst,
        block_table,
        cu_seq_lens,
        token_to_seq,
        total_tokens,
        kv_cache_dtype,
        scale,
        None,
    )
    torch.testing.assert_close(dst, expected)
```
**EN:** This pytest case verifies gather and maybe dequant cache mla. It is parameterized across 9 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lora_rank, qk_rope_head_dim, block_size, num_blocks. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 gather and maybe dequant cache mla 的行为。 它通过 9 组参数化输入覆盖多种场景；它会使用诸如 kv_lora_rank、qk_rope_head_dim、block_size、num_blocks 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_cp_gather_cache_mla` (lines 981-1054)
```python
@pytest.mark.parametrize("kv_lora_rank", [512])
@pytest.mark.parametrize("qk_rope_head_dim", [64])
@pytest.mark.parametrize("block_size", [16])
@pytest.mark.parametrize("num_blocks", [1024])
@pytest.mark.parametrize("max_seq_len", [512])
@pytest.mark.parametrize("batch_size", [8])
@pytest.mark.parametrize("dtype", [torch.float32])
@pytest.mark.parametrize(
    "kv_cache_dtype", ["auto"]
)  # You can also test "fp8" if needed.
@pytest.mark.parametrize("device", CUDA_DEVICES)
@torch.inference_mode()
def test_cp_gather_cache_mla(
    kv_lora_rank,
    qk_rope_head_dim,
    block_size,
    num_blocks,
    max_seq_len,
    batch_size,
    dtype,
    kv_cache_dtype,
    device,
):
    entry_size = kv_lora_rank + qk_rope_head_dim
    src_cache = _create_mla_cache(
        num_blocks, block_size, entry_size, dtype, kv_cache_dtype, device
    )
    _fill_mla_cache(src_cache, kv_cache_dtype=kv_cache_dtype)

    seq_len_tensor = torch.randint(0, max_seq_len + 1, (batch_size,), device=device)

    total_tokens = seq_len_tensor.sum()
    cu_seq_lens = torch.empty((batch_size + 1), dtype=torch.int32, device=device)
    cu_seq_lens[0] = 0
    cu_seq_lens[1:] = seq_len_tensor.cumsum(dim=0).to(dtype=torch.int32)
    print("seq_len_tensor", seq_len_tensor)

    tot_blocks_tensor = (seq_len_tensor + block_size - 1) // block_size
    block_table = torch.empty(
        (batch_size, num_blocks), dtype=torch.int32, device=device
    )

    for b in range(batch_size):
        perm = torch.randperm(num_blocks, device=device)
        block_table[b, :] = perm

    dst = torch.zeros((total_tokens, entry_size), dtype=src_cache.dtype, device=device)

    expected_batches = []
    for b in range(batch_size):
        s = seq_len_tensor[b]
        if s == 0:
            continue
        tot = tot_blocks_tensor[b]
        blocks = block_table[b, :tot].tolist()

        gathered_rows = []
        for i in range(tot - 1):
            gathered_rows.append(src_cache[blocks[i]])
        remaining = s - (tot - 1) * block_size
        gathered_rows.append(src_cache[blocks[-1], :remaining, :])

        batch_expected = torch.cat(gathered_rows, dim=0)
        expected_batches.append(batch_expected)
    expected = torch.cat(expected_batches, dim=0)

    opcheck(
        torch.ops._C_cache_ops.cp_gather_cache,
        (src_cache, dst, block_table, cu_seq_lens, batch_size, None),
        test_utils=DEFAULT_OPCHECK_TEST_UTILS,
    )

    ops.cp_gather_cache(src_cache, dst, block_table, cu_seq_lens, batch_size)
    torch.testing.assert_close(dst, expected)
```
**EN:** This pytest case verifies cp gather cache mla. It is parameterized across 9 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lora_rank, qk_rope_head_dim, block_size, num_blocks. it also validates that the custom operator entry point is wired correctly. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cp gather cache mla 的行为。 它通过 9 组参数化输入覆盖多种场景；它会使用诸如 kv_lora_rank、qk_rope_head_dim、block_size、num_blocks 等 fixture 或输入；它还会校验自定义算子入口是否正确接线；数值结果会在显式容差下与参考结果进行比较。

### Function `test_concat_and_cache_mla_cpu` (lines 1057-1115)
```python
@pytest.mark.parametrize("kv_lora_rank", KV_LORA_RANKS)
@pytest.mark.parametrize("qk_rope_head_dim", QK_ROPE_HEAD_DIMS)
@pytest.mark.parametrize("num_tokens", NUM_TOKENS_MLA)
@pytest.mark.parametrize("block_size", BLOCK_SIZES_MLA)
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS_MLA)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.cpu_model
@pytest.mark.skipif(not current_platform.is_cpu(), reason="CPU only")
@torch.inference_mode()
def test_concat_and_cache_mla_cpu(
    kv_lora_rank: int,
    qk_rope_head_dim: int,
    num_tokens: int,
    block_size: int,
    num_blocks: int,
    dtype: torch.dtype,
    seed: int,
) -> None:
    device = "cpu"
    kv_cache_dtype = "auto"
    set_random_seed(seed)
    torch.set_default_device(device)

    total_slots = num_blocks * block_size
    slot_mapping_lst = random.sample(range(total_slots), num_tokens)
    slot_mapping = torch.tensor(slot_mapping_lst, dtype=torch.long, device=device)

    kv_c = torch.randn(num_tokens, kv_lora_rank, dtype=dtype, device=device)
    k_pe = torch.randn(num_tokens, qk_rope_head_dim, dtype=dtype, device=device)
    entry_size = kv_lora_rank + qk_rope_head_dim

    scale = torch.tensor(0.1, dtype=torch.float32, device=device)
    kv_cache = _create_mla_cache(
        num_blocks, block_size, entry_size, dtype, kv_cache_dtype, device
    )
    ref_temp = torch.zeros(*kv_cache.shape, dtype=dtype, device=device)

    for i in range(num_tokens):
        slot = slot_mapping[i].item()
        block_idx = slot // block_size
        block_offset = slot % block_size
        ref_temp[block_idx, block_offset, :kv_lora_rank] = kv_c[i]
        ref_temp[block_idx, block_offset, kv_lora_rank:] = k_pe[i]

    if kv_cache_dtype == "fp8":
        ref_kv_cache = torch.empty_like(ref_temp, dtype=kv_cache.dtype)
        ops.convert_fp8(ref_kv_cache, ref_temp, scale.item(), kv_dtype=kv_cache_dtype)
    else:
        ref_kv_cache = ref_temp

    opcheck(
        torch.ops._C_cache_ops.concat_and_cache_mla,
        (kv_c, k_pe, kv_cache, slot_mapping, kv_cache_dtype, scale),
        test_utils=DEFAULT_OPCHECK_TEST_UTILS,
    )

    ops.concat_and_cache_mla(kv_c, k_pe, kv_cache, slot_mapping, kv_cache_dtype, scale)
    torch.testing.assert_close(kv_cache, ref_kv_cache)
```
**EN:** This pytest case verifies concat and cache mla CPU. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lora_rank, qk_rope_head_dim, num_tokens, block_size. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly.
**CN:** 该 pytest 用例验证 concat and cache mla CPU 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 kv_lora_rank、qk_rope_head_dim、num_tokens、block_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `random`
- `pytest`
- `torch`
- `tests.kernels.utils -> DEFAULT_OPCHECK_TEST_UTILS, opcheck`
- `vllm -> _custom_ops`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> scaled_dequantize`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> nvfp4_kv_cache_split_views, set_random_seed`
