# test_pos_enc.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_pos_enc.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and module setup
```python
import sys
import time
from typing import Optional, Tuple, Union

import pytest
import torch
import triton
import triton.language as tl

from sglang.jit_kernel.rope import rotary_embedding
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=18, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)


@triton.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 18-33: Function `burn_kernel`
```python
def burn_kernel(out_ptr, iters: tl.constexpr):
    pid = tl.program_id(0)
    x = tl.full((), pid + 1, dtype=tl.uint32)

    a = tl.full((), 1664525, dtype=tl.uint32)
    c = tl.full((), 1013904223, dtype=tl.uint32)
    sh = tl.full((), 13, dtype=tl.uint32)

    for _ in range(iters):
        x = x * a + c
        x = x ^ (x >> sh)

    if pid == 0:
        tl.store(out_ptr, x)
```
**EN:** This block defines `burn_kernel`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `burn_kernel`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 34-40: Function `triton_burn`
```python
def triton_burn(ms: float, grid=(256,)):
    iters = int(ms * 20000)
    out = torch.empty((), device="cuda", dtype=torch.uint32)
    burn_kernel[grid](out, iters=iters)
    return out
```
**EN:** This block defines `triton_burn`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `triton_burn`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 41-63: Function `create_test_inputs`
```python
def create_test_inputs(
    head_size, batch_size, seq_len, device, dtype, num_q_heads, num_kv_heads
):
    """Create test inputs."""
    total_tokens = batch_size * seq_len

    query = torch.randn(
        batch_size, seq_len, num_q_heads, head_size, dtype=dtype, device=device
    )
    key = torch.randn(
        batch_size, seq_len, num_kv_heads, head_size, dtype=dtype, device=device
    )

    pos_ids = torch.randint(
        0, min(seq_len * 2, 100), (total_tokens,), dtype=torch.long, device=device
    )

    query = query.view(total_tokens, num_q_heads, head_size)
    key = key.view(total_tokens, num_kv_heads, head_size)

    return query, key, pos_ids
```
**EN:** This block defines `create_test_inputs`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `create_test_inputs`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 64-90: Function `create_cos_sin_cache`
```python
def create_cos_sin_cache(rotary_dim, max_position_embeddings, base, dtype, device):
    """Create cos/sin cache for rotary embedding."""
    max_pos = max_position_embeddings
    extended_max_pos = max(max_pos, 100)
    cos_sin_cache = torch.zeros(
        extended_max_pos, rotary_dim, dtype=dtype, device=device
    )

    inv_freq = 1.0 / (
        base
        ** (
            torch.arange(0, rotary_dim, 2, dtype=torch.float32, device=device)
            / rotary_dim
        )
    )
    t = torch.arange(extended_max_pos, dtype=torch.float32, device=device)
    freqs = torch.outer(t, inv_freq)
    cos_cache = torch.cos(freqs).to(dtype)
    sin_cache = torch.sin(freqs).to(dtype)

    cos_sin_cache[:, : rotary_dim // 2] = cos_cache
    cos_sin_cache[:, rotary_dim // 2 :] = sin_cache

    return cos_sin_cache


# vLLM torch native
```
**EN:** This block defines `create_cos_sin_cache`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `create_cos_sin_cache`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 91-119: Function `_apply_rotary_emb`
```python
def _apply_rotary_emb(
    x: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
    is_neox_style: bool,
) -> torch.Tensor:
    """
    Args:
        x: [num_tokens, num_heads, head_size]
        cos: [num_tokens, head_size // 2]
        sin: [num_tokens, head_size // 2]
        is_neox_style: Whether to use the Neox-style or GPT-J-style rotary
            positional embeddings.
    """
    cos = cos.unsqueeze(-2).to(x.dtype)
    sin = sin.unsqueeze(-2).to(x.dtype)
    if is_neox_style:
        x1, x2 = torch.chunk(x, 2, dim=-1)
    else:
        x1 = x[..., ::2]
        x2 = x[..., 1::2]
    o1 = x1 * cos - x2 * sin
    o2 = x2 * cos + x1 * sin
    if is_neox_style:
        return torch.cat((o1, o2), dim=-1)
    else:
        return torch.stack((o1, o2), dim=-1).flatten(-2)
```
**EN:** This block defines `_apply_rotary_emb`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_apply_rotary_emb`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 120-203: Class `RotaryEmbedding`
```python
class RotaryEmbedding(torch.nn.Module):
    # Reference: https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/layers/rotary_embedding.py
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        dtype: torch.dtype,
    ) -> None:
        super().__init__()
        self.head_size = head_size
        self.rotary_dim = rotary_dim
        self.max_position_embeddings = max_position_embeddings
        self.base = base
        self.is_neox_style = is_neox_style
        self.dtype = dtype

        cache = self._compute_cos_sin_cache()
        self.cos_sin_cache: torch.Tensor
        self.register_buffer("cos_sin_cache", cache, persistent=False)

    def _compute_inv_freq(self, base: Union[int, float]) -> torch.Tensor:
        inv_freq = 1.0 / (
            base
            ** (
                torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
            )
        )
        return inv_freq

    def _compute_cos_sin_cache(self) -> torch.Tensor:
        """Compute the cos and sin cache."""
        inv_freq = self._compute_inv_freq(self.base)
        t = torch.arange(self.max_position_embeddings, dtype=torch.float)

        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos()
        sin = freqs.sin()
        cache = torch.cat((cos, sin), dim=-1)
        return cache

    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: Optional[torch.Tensor] = None,
        offsets: Optional[torch.Tensor] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """A PyTorch-native implementation of forward()."""

        if offsets is not None:
            positions = positions + offsets

        positions = positions.flatten()
        num_tokens = positions.shape[0]
        cos_sin = self.cos_sin_cache.index_select(0, positions)

        cos, sin = cos_sin.chunk(2, dim=-1)

        query_shape = query.shape
        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = _apply_rotary_emb(query_rot, cos, sin, self.is_neox_style)
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        # Modification: convert to the correct dtype
        query = query.to(self.dtype)

        if key is not None:
            key_shape = key.shape
            key = key.view(num_tokens, -1, self.head_size)
            key_rot = key[..., : self.rotary_dim]
            key_pass = key[..., self.rotary_dim :]
            key_rot = _apply_rotary_emb(key_rot, cos, sin, self.is_neox_style)
            key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)

            key = key.to(self.dtype)
# ...
```
**EN:** This block declares the `RotaryEmbedding` class and establishes the behavior or state it encapsulates. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `RotaryEmbedding` 类，并建立其封装的行为或状态。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 204-217: Function `get_torch_rotary_embedding`
```python
def get_torch_rotary_embedding(
    head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype, device
):
    """Initialize Torch Native RotaryEmbedding based on vLLM implementation."""
    return RotaryEmbedding(
        head_size=head_size,
        rotary_dim=rotary_dim,
        max_position_embeddings=max_position_embeddings,
        base=base,
        is_neox_style=is_neox_style,
        dtype=dtype,
    ).to(device)
```
**EN:** This block defines `get_torch_rotary_embedding`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `get_torch_rotary_embedding`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 218-238: Imports and module setup
```python
def get_sgl_rotary_embedding(
    head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype, device
):
    """Initialize SglKernelRotaryEmbedding."""
    try:
        from sgl_kernel.testing.rotary_embedding import SglKernelRotaryEmbedding
    except ImportError:
        pytest.skip(
            "SglKernelRotaryEmbedding is not available. Test case can be removed."
        )

    return SglKernelRotaryEmbedding(
        head_size=head_size,
        rotary_dim=rotary_dim,
        max_position_embeddings=max_position_embeddings,
        base=base,
        is_neox_style=is_neox_style,
        dtype=dtype,
    ).to(device)
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 239-288: Function `compare_results`
```python
def compare_results(jit_out, sgl_out, dtype):
    """Compare results between JIT and SGL implementations."""
    if jit_out is None:
        assert sgl_out is None
        return

    assert sgl_out is not None

    # Check for NaN values
    assert not torch.isnan(jit_out).any(), "NaN in JIT results"
    assert not torch.isnan(sgl_out).any(), "NaN in SGL results"

    # Compare results
    atol = 4e-2 if dtype != torch.float32 else 1e-5
    rtol = 4e-2 if dtype != torch.float32 else 1e-5

    torch.testing.assert_close(jit_out, sgl_out, atol=atol, rtol=rtol)


@pytest.mark.parametrize(
    "head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype, device, batch_size, seq_len, num_q_heads, num_kv_heads",
    [
        # GPT-OSS cases
        *[
            (64, 64, 4096, 8000, True, torch.bfloat16, "cuda", bs, sl, 8, 8)
            for bs, sl in [(1, 1), (32, 1), (128, 1), (512, 1), (2, 512), (4, 4096)]
        ],
        # Other cases
        (64, 64, 32, 8000, True, torch.bfloat16, "cuda", 32, 32, 1, 1),
        (256, 128, 4096, 10000, True, torch.bfloat16, "cuda", 2, 512, 4, 2),
        (512, 128, 311, 10000, True, torch.bfloat16, "cuda", 3, 39, 4, 2),
        (128, 128, 2048, 10000, False, torch.bfloat16, "cuda", 2, 512, 32, 8),
        (128, 128, 2048, 10000, False, torch.bfloat16, "cuda", 2, 512, 16, 4),
        (512, 128, 311, 10000, False, torch.bfloat16, "cuda", 3, 39, 4, 2),
        (64, 64, 32, 8000, True, torch.float32, "cuda", 32, 32, 1, 1),
        (256, 128, 4096, 10000, True, torch.float32, "cuda", 2, 512, 4, 2),
        (512, 128, 311, 10000, True, torch.float32, "cuda", 3, 39, 4, 2),
        (128, 128, 2048, 10000, False, torch.float32, "cuda", 2, 512, 32, 8),
        (128, 128, 2048, 10000, False, torch.float32, "cuda", 2, 512, 16, 4),
        (512, 128, 311, 10000, False, torch.float32, "cuda", 3, 39, 4, 2),
        # Additional test cases for different head sizes and dtypes
        (64, 32, 1024, 10000, True, torch.float16, "cuda", 16, 64, 8, 4),
        (128, 64, 2048, 10000, True, torch.float16, "cuda", 8, 128, 16, 8),
        (256, 128, 4096, 10000, True, torch.float16, "cuda", 4, 256, 8, 4),
    ],
)
@pytest.mark.parametrize(
    "key_is_none",
    [True, False],
)
```
**EN:** This block defines `compare_results`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `compare_results`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 289-377: Function `test_correctness`
```python
def test_correctness(
    head_size,
    rotary_dim,
    max_position_embeddings,
    base,
    is_neox_style,
    dtype,
    device,
    batch_size,
    seq_len,
    num_q_heads,
    num_kv_heads,
    key_is_none,
):
    """Test correctness of JIT rotary embedding implementation."""
    # Create inputs and caches
    query, key, pos_ids = create_test_inputs(
        head_size, batch_size, seq_len, device, dtype, num_q_heads, num_kv_heads
    )
    cos_sin_cache = create_cos_sin_cache(
        rotary_dim, max_position_embeddings, base, dtype, device
    )

    # Initialize torch kernel
    torch_rotary_emb = get_torch_rotary_embedding(
        head_size,
        rotary_dim,
        max_position_embeddings,
        base,
        is_neox_style,
        dtype,
        device,
    )
    torch_rotary_emb.cos_sin_cache = cos_sin_cache
    r = torch.randn_like(query)

    # Apply rotary embeddings
    query_jit, key_jit = query.clone(), key.clone()
    query_torch, key_torch = query.clone(), key.clone()
    stream_jit = torch.get_device_module("cuda").Stream()
    stream_kernel = torch.get_device_module("cuda").Stream()

    if key_is_none:
        key_jit = None
        key_torch = None
    triton_burn(100.0, grid=(1024,))

    r_jit, r_torch = r.clone(), r.clone()
    torch.cuda.synchronize()

    with torch.cuda.stream(stream_jit):
        # Test if rotary_embedding runs on stream_jit
        triton_burn(100.0, grid=(1024,))
        query_jit = query_jit + r_jit
        query_jit_out, key_jit_out = rotary_embedding(
            positions=pos_ids,
            query=query_jit,
            key=key_jit,
            head_size=head_size,
            cos_sin_cache=cos_sin_cache,
            is_neox=is_neox_style,
        )

    with torch.cuda.stream(stream_kernel):
        triton_burn(100.0, grid=(1024,))
        query_torch = query_torch + r_torch
        query_torch_out, key_torch_out = torch_rotary_emb.forward_native(
            positions=pos_ids, query=query_torch, key=key_torch
        )

    torch.cuda.synchronize()
    compare_results(query_jit_out, query_torch_out, dtype)
    compare_results(key_jit_out, key_torch_out, dtype)


@pytest.mark.parametrize(
    "head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype, device, batch_size, seq_len, num_q_heads, num_kv_heads",
    [
        # Small scale
        (64, 64, 4096, 8000, True, torch.bfloat16, "cuda", 1, 1, 8, 8),
# ...
```
**EN:** This block defines `test_correctness`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_correctness`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 378-494: Function `test_performance`
```python
def test_performance(
    head_size: int,
    rotary_dim: int,
    max_position_embeddings: int,
    base: int,
    is_neox_style,
    dtype,
    device,
    batch_size,
    seq_len,
    num_q_heads,
    num_kv_heads,
):
    """Performance test comparing JIT and SGL implementations with accuracy validation."""
    # Create inputs and caches
    query, key, pos_ids = create_test_inputs(
        head_size, batch_size, seq_len, device, dtype, num_q_heads, num_kv_heads
    )
    cos_sin_cache = create_cos_sin_cache(
        rotary_dim, max_position_embeddings, base, dtype, device
    )

    # Initialize SGL kernel
    sgl_rotary_emb = get_sgl_rotary_embedding(
        head_size,
        rotary_dim,
        max_position_embeddings,
        base,
        is_neox_style,
        dtype,
        device,
    )
    sgl_rotary_emb.cos_sin_cache = cos_sin_cache

    warmup = 3

    # Warmup runs
    for _ in range(warmup):
        query_warm, key_warm = query.clone(), key.clone()
        rotary_embedding(
            positions=pos_ids,
            query=query_warm,
            key=key_warm,
            head_size=head_size,
            cos_sin_cache=cos_sin_cache,
            is_neox=is_neox_style,
        )

        query_sgl_warm, key_sgl_warm = query.clone(), key.clone()
        sgl_rotary_emb.forward_cuda(
            positions=pos_ids, query=query_sgl_warm, key=key_sgl_warm
        )

    iteration = 100

    # Time JIT implementation
    torch.cuda.synchronize()
    start_time = time.time()
    for _ in range(iteration):
        query_jit, key_jit = query.clone(), key.clone()
        rotary_embedding(
            positions=pos_ids,
            query=query_jit,
            key=key_jit,
            head_size=head_size,
            cos_sin_cache=cos_sin_cache,
            is_neox=is_neox_style,
        )
    torch.cuda.synchronize()
    jit_time = (time.time() - start_time) / iteration

    # Time SGL implementation
    torch.cuda.synchronize()
    start_time = time.time()
    for _ in range(iteration):
        query_sgl, key_sgl = query.clone(), key.clone()
        sgl_rotary_emb.forward_cuda(positions=pos_ids, query=query_sgl, key=key_sgl)
    torch.cuda.synchronize()
    sgl_time = (time.time() - start_time) / iteration

# ...
```
**EN:** This block defines `test_performance`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_performance`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `time`
- `typing -> Optional`
- `pytest`
- `torch`
- `triton`
- `triton.language as tl`
- `sglang.jit_kernel.rope -> rotary_embedding`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sgl_kernel.testing.rotary_embedding -> SglKernelRotaryEmbedding`
