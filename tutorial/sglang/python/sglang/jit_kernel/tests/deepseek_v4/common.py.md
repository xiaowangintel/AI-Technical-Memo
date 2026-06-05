# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/deepseek_v4/common.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and module setup
```python
from __future__ import annotations

from dataclasses import dataclass
from typing import List, Literal, Optional, Tuple

import torch

from sglang.jit_kernel.dsv4 import CompressorDecodePlan, CompressorPrefillPlan


@dataclass
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 12-60: Class `LegacyContext`
```python
class LegacyContext:
    """Per-request ring buffer (no req_to_token / full_to_swa).

    `req_pool_indices[i]` directly maps to the request's ring base slot.
    """

    bs: int
    head_dim: int
    compress_ratio: int
    req_pool_indices: torch.Tensor  # int64 [bs] on cuda
    pages_per_req: int

    @property
    def num_pages(self) -> int:
        # Reserve enough pages to hold all batched requests' rings.
        return int(self.req_pool_indices.max().item() + 1) * self.pages_per_req

    def state_loc(self, b: int, position: int) -> int:
        rid = int(self.req_pool_indices[b].item())
        if self.compress_ratio == 4:
            page = rid * 2 + (position // 4) % 2
        else:
            page = rid
        return page * self.compress_ratio + position % self.compress_ratio

    def make_prefill_plan(
        self,
        seq_lens_cpu: torch.Tensor,
        extend_lens_cpu: torch.Tensor,
        num_q_tokens: int,
    ) -> CompressorPrefillPlan:
        return CompressorPrefillPlan.generate_legacy(
            compress_ratio=self.compress_ratio,  # type: ignore
            req_pool_indices=self.req_pool_indices,
            seq_lens=seq_lens_cpu,
            extend_lens=extend_lens_cpu,
            num_q_tokens=num_q_tokens,
            device=torch.device("cuda"),
        )

    def make_decode_plan(self, seq_lens_gpu: torch.Tensor) -> CompressorDecodePlan:
        return CompressorDecodePlan.generate_legacy(
            compress_ratio=self.compress_ratio,  # type: ignore
            req_pool_indices=self.req_pool_indices,
            seq_lens=seq_lens_gpu,
        )


@dataclass
```
**EN:** This block declares the `LegacyContext` class and establishes the behavior or state it encapsulates. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `LegacyContext` 类，并建立其封装的行为或状态。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 61-123: Class `PagedContext`
```python
class PagedContext:
    """SWA paged layout with identity req_to_token + identity full_to_swa.

    Each request occupies `num_swa_pages_per_req` contiguous swa_pages, so
    `req_to_token[r, p] = r * (num_swa_pages_per_req * swa_page_size) + p`.
    """

    bs: int
    head_dim: int
    compress_ratio: int
    swa_page_size: int
    ring_size: int
    num_swa_pages_per_req: int
    req_pool_indices: torch.Tensor  # int64 [bs] on cuda
    req_to_token: torch.Tensor  # int64 [num_reqs_capacity, max_tokens_per_req] on cuda
    full_to_swa: torch.Tensor  # int64 [num_swa_slots] on cuda

    @property
    def num_pages(self) -> int:
        # Upper bound: every (request, position) state slot fits.
        max_state_loc = (
            self.bs * self.num_swa_pages_per_req * self.ring_size
            + self.swa_page_size  # slack for the largest tail
        )
        return max_state_loc // self.compress_ratio + 1

    def state_loc(self, b: int, position: int) -> int:
        rid = int(self.req_pool_indices[b].item())
        loc = int(self.req_to_token[rid, position].item())
        swa_loc = int(self.full_to_swa[loc].item())
        swa_page = swa_loc // self.swa_page_size
        return swa_page * self.ring_size + swa_loc % self.ring_size

    def make_prefill_plan(
        self,
        seq_lens_cpu: torch.Tensor,
        extend_lens_cpu: torch.Tensor,
        num_q_tokens: int,
    ) -> CompressorPrefillPlan:
        return CompressorPrefillPlan.generate(
            compress_ratio=self.compress_ratio,  # type: ignore
            req_pool_indices=self.req_pool_indices,
            seq_lens=seq_lens_cpu,
            extend_lens=extend_lens_cpu,
            req_to_token=self.req_to_token,
            full_to_swa=self.full_to_swa,
            swa_page_size=self.swa_page_size,
            ring_size=self.ring_size,
            num_q_tokens=num_q_tokens,
        )

    def make_decode_plan(self, seq_lens_gpu: torch.Tensor) -> CompressorDecodePlan:
        return CompressorDecodePlan.generate(
            compress_ratio=self.compress_ratio,  # type: ignore
            req_pool_indices=self.req_pool_indices,
            req_to_token=self.req_to_token,
            full_to_swa=self.full_to_swa,
            seq_lens=seq_lens_gpu,
            swa_page_size=self.swa_page_size,
            ring_size=self.ring_size,
        )
```
**EN:** This block declares the `PagedContext` class and establishes the behavior or state it encapsulates. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `PagedContext` 类，并建立其封装的行为或状态。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 124-139: Function `make_legacy_context`
```python
def make_legacy_context(
    bs: int,
    compress_ratio: Literal[4, 128],
    head_dim: int = 512,
) -> LegacyContext:
    pages_per_req = 2 if compress_ratio == 4 else 1
    req_pool_indices = torch.arange(bs, dtype=torch.int64, device="cuda")
    return LegacyContext(
        bs=bs,
        head_dim=head_dim,
        compress_ratio=compress_ratio,
        req_pool_indices=req_pool_indices,
        pages_per_req=pages_per_req,
    )
```
**EN:** This block defines `make_legacy_context`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `make_legacy_context`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 140-179: Function `make_paged_context`
```python
def make_paged_context(
    bs: int,
    compress_ratio: Literal[4, 128],
    head_dim: int = 512,
    swa_page_size: int = 256,
    ring_size: Optional[int] = None,
    num_swa_pages_per_req: int = 8,
    max_tokens_per_req: int = 8192,
    num_reqs_capacity: int = 16,
) -> PagedContext:
    if ring_size is None:
        ring_size = 8 if compress_ratio == 4 else 128
    assert swa_page_size % ring_size == 0
    assert ring_size % compress_ratio == 0
    assert num_swa_pages_per_req * swa_page_size <= max_tokens_per_req

    stride = num_swa_pages_per_req * swa_page_size
    req_to_token = torch.zeros(
        (num_reqs_capacity, max_tokens_per_req), dtype=torch.int32
    )
    for r in range(bs):
        req_to_token[r, :stride] = torch.arange(
            r * stride, (r + 1) * stride, dtype=torch.int32
        )
    total_swa_slots = num_reqs_capacity * stride
    full_to_swa = torch.arange(total_swa_slots, dtype=torch.int64)
    req_pool_indices = torch.arange(bs, dtype=torch.int64)
    return PagedContext(
        bs=bs,
        head_dim=head_dim,
        compress_ratio=compress_ratio,
        swa_page_size=swa_page_size,
        ring_size=ring_size,
        num_swa_pages_per_req=num_swa_pages_per_req,
        req_pool_indices=req_pool_indices.cuda(),
        req_to_token=req_to_token.cuda(),
        full_to_swa=full_to_swa.cuda(),
    )
```
**EN:** This block defines `make_paged_context`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `make_paged_context`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 180-188: Function `make_state_pool`
```python
def make_state_pool(num_pages: int, compress_ratio: int, head_dim: int) -> torch.Tensor:
    last_dim = head_dim * (4 if compress_ratio == 4 else 2)
    return torch.zeros(
        (num_pages, compress_ratio, last_dim),
        dtype=torch.float32,
        device="cuda",
    )
```
**EN:** This block defines `make_state_pool`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `make_state_pool`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 189-195: Function `to_seq_extend`
```python
def to_seq_extend(
    seq_extend_pairs: List[Tuple[int, int]],
) -> Tuple[torch.Tensor, torch.Tensor, int]:
    seq_lens = torch.tensor([s for s, _ in seq_extend_pairs], dtype=torch.int64)
    extend_lens = torch.tensor([e for _, e in seq_extend_pairs], dtype=torch.int64)
    num_q = int(extend_lens.sum().item())
    return seq_lens, extend_lens, num_q
```
**EN:** This block defines `to_seq_extend`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `to_seq_extend`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `dataclasses -> dataclass`
- `typing -> List`
- `torch`
- `sglang.jit_kernel.dsv4 -> CompressorDecodePlan`
