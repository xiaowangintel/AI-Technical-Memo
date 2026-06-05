# test_c128_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/deepseek_v4/test_c128_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and module setup
```python
from __future__ import annotations

import sys
from typing import Tuple, Union

import pytest
import torch
import triton

from sglang.jit_kernel.benchmark.bench_activation import register_cuda_ci
from sglang.jit_kernel.dsv4 import compress_forward
from sglang.jit_kernel.tests.deepseek_v4.common import (
    LegacyContext,
    PagedContext,
    make_legacy_context,
    make_paged_context,
    make_state_pool,
    to_seq_extend,
)

register_cuda_ci(est_time=30, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=30, suite="nightly-kernel-1-gpu", nightly=True)

Context = Union[LegacyContext, PagedContext]

# c128 input row layout: | kv | score |  each [head_dim]
HEAD_DIM = 512
RATIO = 128
ATOL = 5e-3
RTOL = 5e-3
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 33-45: Function `_gt_compress`
```python
def _gt_compress(
    kv_score_input_cpu: torch.Tensor,  # [num_q, head_dim*2]
    ape_cpu: torch.Tensor,  # [128, head_dim]
    P: int,
    head_dim: int,
) -> torch.Tensor:
    """fp64 reference for compress event at ragged position ``P`` (P % 128 == 127)."""
    lo = P - (RATIO - 1)
    kv = kv_score_input_cpu[lo : P + 1, :head_dim].double()
    sc = kv_score_input_cpu[lo : P + 1, head_dim:].double()
    return ((kv * (sc + ape_cpu.double()).softmax(dim=0)).sum(dim=0)).float()
```
**EN:** This block defines `_gt_compress`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_gt_compress`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 46-56: Function `_make_inputs`
```python
def _make_inputs(
    num_q: int, head_dim: int, seed: int
) -> Tuple[torch.Tensor, torch.Tensor]:
    g = torch.Generator(device="cpu").manual_seed(seed)
    kv_score_input_cpu = torch.randn(
        num_q, head_dim * 2, generator=g, dtype=torch.float32
    )
    ape_cpu = torch.randn(RATIO, head_dim, generator=g, dtype=torch.float32)
    return kv_score_input_cpu, ape_cpu
```
**EN:** This block defines `_make_inputs`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_make_inputs`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 57-76: Function `_run_prefill`
```python
def _run_prefill(
    ctx: Context,
    pool: torch.Tensor,
    kv_score_input: torch.Tensor,
    ape: torch.Tensor,
    seq_lens_cpu: torch.Tensor,
    extend_lens_cpu: torch.Tensor,
) -> torch.Tensor:
    num_q = int(extend_lens_cpu.sum().item())
    plan = ctx.make_prefill_plan(seq_lens_cpu, extend_lens_cpu, num_q)
    return compress_forward(
        pool,
        kv_score_input,
        ape,
        plan,
        head_dim=ctx.head_dim,
        compress_ratio=RATIO,
    )
```
**EN:** This block defines `_run_prefill`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_run_prefill`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 77-101: Function `_run_decode`
```python
def _run_decode(
    ctx: Context,
    pool: torch.Tensor,
    kv_score_input: torch.Tensor,
    ape: torch.Tensor,
    seq_lens_gpu: torch.Tensor,
) -> torch.Tensor:
    plan = ctx.make_decode_plan(seq_lens_gpu)
    return compress_forward(
        pool,
        kv_score_input,
        ape,
        plan,
        head_dim=ctx.head_dim,
        compress_ratio=RATIO,
    )


# -----------------------------------------------------------------------------
# Tests
# -----------------------------------------------------------------------------


@pytest.mark.parametrize("mode", ["legacy", "paged"])
@pytest.mark.parametrize("seq_len", [128, 256, 512])
```
**EN:** This block defines `_run_decode`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_run_decode`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 102-126: Function `test_prefill_no_context`
```python
def test_prefill_no_context(mode: str, seq_len: int) -> None:
    """Single-shot prefill, no prefix. Every compress event must match fp64 GT."""
    if mode == "legacy":
        ctx: Context = make_legacy_context(
            bs=1, compress_ratio=RATIO, head_dim=HEAD_DIM
        )
    else:
        ctx = make_paged_context(bs=1, compress_ratio=RATIO, head_dim=HEAD_DIM)

    seq_lens_cpu, extend_lens_cpu, num_q = to_seq_extend([(seq_len, seq_len)])
    kv_in_cpu, ape_cpu = _make_inputs(num_q, ctx.head_dim, seed=seq_len)

    pool = make_state_pool(ctx.num_pages, RATIO, ctx.head_dim)
    out = _run_prefill(
        ctx, pool, kv_in_cpu.cuda(), ape_cpu.cuda(), seq_lens_cpu, extend_lens_cpu
    )

    # Compact prefill output: row per compress plan, in CPU-planner order.
    for plan_id, P in enumerate(range(RATIO - 1, seq_len, RATIO)):
        gt = _gt_compress(kv_in_cpu, ape_cpu, P=P, head_dim=ctx.head_dim)
        triton.testing.assert_close(out[plan_id].cpu(), gt, atol=ATOL, rtol=RTOL)


@pytest.mark.parametrize("mode", ["legacy", "paged"])
@pytest.mark.parametrize("prefix_len", [0, 128, 256])
```
**EN:** This block defines `test_prefill_no_context`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_prefill_no_context`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 127-170: Function `test_prefill_then_decode`
```python
def test_prefill_then_decode(mode: str, prefix_len: int) -> None:
    """Prefill ``prefix_len`` tokens, then decode through to the next 128 boundary."""
    seq_len = prefix_len + RATIO  # one full compress chunk after prefix

    if mode == "legacy":
        ctx: Context = make_legacy_context(
            bs=1, compress_ratio=RATIO, head_dim=HEAD_DIM
        )
    else:
        ctx = make_paged_context(bs=1, compress_ratio=RATIO, head_dim=HEAD_DIM)

    kv_full_cpu, ape_cpu = _make_inputs(
        seq_len, ctx.head_dim, seed=seq_len + prefix_len
    )
    pool = make_state_pool(ctx.num_pages, RATIO, ctx.head_dim)

    if prefix_len > 0:
        seq_lens_cpu, extend_lens_cpu, _ = to_seq_extend([(prefix_len, prefix_len)])
        _run_prefill(
            ctx,
            pool,
            kv_full_cpu[:prefix_len].cuda(),
            ape_cpu.cuda(),
            seq_lens_cpu,
            extend_lens_cpu,
        )

    final_out = None
    for k in range(RATIO):
        cur_seq_len = prefix_len + k + 1
        seq_lens_gpu = torch.tensor([cur_seq_len], dtype=torch.int64, device="cuda")
        kv_step = kv_full_cpu[prefix_len + k : prefix_len + k + 1].cuda()
        out = _run_decode(ctx, pool, kv_step, ape_cpu.cuda(), seq_lens_gpu)
        if cur_seq_len % RATIO == 0:
            final_out = out

    P = seq_len - 1
    gt = _gt_compress(kv_full_cpu, ape_cpu, P=P, head_dim=ctx.head_dim)
    assert final_out is not None
    triton.testing.assert_close(final_out[0].cpu(), gt, atol=ATOL, rtol=RTOL)


@pytest.mark.parametrize("mode", ["legacy", "paged"])
@pytest.mark.parametrize("prefix_len", [128, 256])
```
**EN:** This block defines `test_prefill_then_decode`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_prefill_then_decode`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 171-215: Function `test_prefill_then_extend`
```python
def test_prefill_then_extend(mode: str, prefix_len: int) -> None:
    """Prefill once, then a second prefill that extends across one compress event.

    First prefill ends at a 128-boundary so the second prefill starts fresh.
    """
    extend_len = RATIO
    seq_len = prefix_len + extend_len

    if mode == "legacy":
        ctx: Context = make_legacy_context(
            bs=1, compress_ratio=RATIO, head_dim=HEAD_DIM
        )
    else:
        ctx = make_paged_context(bs=1, compress_ratio=RATIO, head_dim=HEAD_DIM)

    kv_full_cpu, ape_cpu = _make_inputs(seq_len, ctx.head_dim, seed=prefix_len)
    pool = make_state_pool(ctx.num_pages, RATIO, ctx.head_dim)

    seq_lens_cpu, extend_lens_cpu, _ = to_seq_extend([(prefix_len, prefix_len)])
    _run_prefill(
        ctx,
        pool,
        kv_full_cpu[:prefix_len].cuda(),
        ape_cpu.cuda(),
        seq_lens_cpu,
        extend_lens_cpu,
    )

    seq_lens_cpu, extend_lens_cpu, _ = to_seq_extend([(seq_len, extend_len)])
    out = _run_prefill(
        ctx,
        pool,
        kv_full_cpu[prefix_len:].cuda(),
        ape_cpu.cuda(),
        seq_lens_cpu,
        extend_lens_cpu,
    )

    P = seq_len - 1
    gt = _gt_compress(kv_full_cpu, ape_cpu, P=P, head_dim=ctx.head_dim)
    # Single compress event in this extend; compact plan_id 0.
    triton.testing.assert_close(out[0].cpu(), gt, atol=ATOL, rtol=RTOL)


@pytest.mark.parametrize("mode", ["legacy", "paged"])
```
**EN:** This block defines `test_prefill_then_extend`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_prefill_then_extend`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 216-260: Function `test_prefill_multibatch`
```python
def test_prefill_multibatch(mode: str) -> None:
    """Multi-batch prefill, each batch ending at a different chunk count."""
    seq_extend = [(128, 128), (256, 256), (384, 384)]
    bs = len(seq_extend)
    if mode == "legacy":
        ctx: Context = make_legacy_context(
            bs=bs, compress_ratio=RATIO, head_dim=HEAD_DIM
        )
    else:
        ctx = make_paged_context(bs=bs, compress_ratio=RATIO, head_dim=HEAD_DIM)

    seq_lens_cpu, extend_lens_cpu, num_q = to_seq_extend(seq_extend)
    kv_in_cpu, ape_cpu = _make_inputs(num_q, ctx.head_dim, seed=99)
    pool = make_state_pool(ctx.num_pages, RATIO, ctx.head_dim)
    out = _run_prefill(
        ctx, pool, kv_in_cpu.cuda(), ape_cpu.cuda(), seq_lens_cpu, extend_lens_cpu
    )

    # Compact: walk batches in order, then positions in order; matches the
    # CPU planner's emit order for plan_c.
    base = 0
    plan_id = 0
    for b, (seq, ext) in enumerate(seq_extend):
        for j in range(ext):
            P = j  # prefix=0
            if (P + 1) % RATIO != 0:
                continue
            gt = _gt_compress(
                kv_in_cpu[base : base + ext],
                ape_cpu,
                P=P,
                head_dim=ctx.head_dim,
            )
            triton.testing.assert_close(
                out[plan_id].cpu(),
                gt,
                atol=ATOL,
                rtol=RTOL,
            )
            plan_id += 1
        base += ext


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This block defines `test_prefill_multibatch`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_prefill_multibatch`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `sys`
- `typing -> Tuple`
- `pytest`
- `torch`
- `triton`
- `sglang.jit_kernel.benchmark.bench_activation -> register_cuda_ci`
- `sglang.jit_kernel.dsv4 -> compress_forward`
- `sglang.jit_kernel.tests.deepseek_v4.common -> (`
