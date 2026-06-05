# _paged_attention.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/attention/experimental/_paged_attention.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements attention-specific abstractions and helpers for modern sequence models.
- **Purpose (CN)**: 实现现代序列模型中的注意力相关抽象与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```python
# mypy: allow-untyped-defs
"""
This module implements Paged Attention on top of flex_attention.
This module is experimental and subject to change.
"""

import torch
from torch.nn.attention.flex_attention import (
    _identity,
    _mask_mod_signature,
    _score_mod_signature,
    BlockMask,
    noop_mask,
)


__all__ = ["PagedAttention"]


def _cdiv(x: int | float | torch.Tensor, multiple: int | float | torch.Tensor):
    return (x + multiple - 1) // multiple
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 24-50
```python
class PagedAttention:
    """
    PagedAttention supports flex attention inference with a large batch size.
    With PagedAttention, a batch of key/value tensors with varying kv length
    is split into tensor blocks of fixed length and cached in a compact way.
    Thus we can avoid redundant memory consumption due to varying kv length and
    support a larger batch size.
    """

    def __init__(
        self,
        n_pages: int,
        page_size: int,
        max_batch_size: int,
        device: str = "cuda",
    ) -> None:
        # number of pages
        self.n_pages = n_pages

        # number of tokens per page
        self.page_size = page_size

        # page table: [batch, logical_block_idx] -> physical_page_idx
        self.page_table = -torch.ones(
            (max_batch_size, self.n_pages), dtype=torch.int64, device=device
        )
```
- **EN**: Declares `PagedAttention`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `PagedAttention`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 51-74
```python
        # capacity: batch_idx -> allocated sequence length
        self.capacity = torch.zeros(max_batch_size, dtype=torch.int64, device=device)

        # index of empty pages that is available for allocation
        self.empty_pages = list(range(n_pages - 1, -1, -1))

        # mapping from physical page index to logical page index
        self.physical_to_logical = -torch.ones(
            (max_batch_size, n_pages), dtype=torch.int64, device=device
        )

    def reserve(self, batch_idx: torch.Tensor, seq_len: torch.Tensor) -> None:
        """
        Requests the capacity of a given batch to be at least enough to
        hold `seq_len` elements.

        Args:
            batch_idx (Tensor): batch index to be reserved; shape :math:`(1)`.
            seq_len (Tensor): minimum capacity for the given batch; shape :math:`(1)`.
        """

        if seq_len <= self.capacity[batch_idx]:
            return
```
- **EN**: Declares `PagedAttention`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `PagedAttention`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 75-100
```python
        num_pages_to_allocate = _cdiv(
            seq_len - self.capacity[batch_idx], self.page_size
        )

        if len(self.empty_pages) < num_pages_to_allocate:
            raise AssertionError(
                f"requested {num_pages_to_allocate.item()} pages "
                f"but there are only {len(self.empty_pages)} empty pages"
            )

        start_page_idx = self.capacity[batch_idx] // self.page_size
        end_page_idx = start_page_idx + num_pages_to_allocate

        # find empty physical pages
        allocated_pages = torch.tensor(
            self.empty_pages[-num_pages_to_allocate:],
            device=num_pages_to_allocate.device,
        )
        self.empty_pages = self.empty_pages[:-num_pages_to_allocate]

        # update page table
        self.page_table[
            batch_idx,
            start_page_idx:end_page_idx,
        ] = allocated_pages
```
- **EN**: This block continues `PagedAttention.reserve` and works to normalize dtype/device related arguments and behavior. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `PagedAttention.reserve`，用于规范化 dtype/device 相关参数与行为。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 101-126
```python
        # update metadata
        self.physical_to_logical[batch_idx, allocated_pages] = torch.arange(
            start_page_idx.item(),
            end_page_idx.item(),
            device=num_pages_to_allocate.device,
        )
        self.capacity[batch_idx] += num_pages_to_allocate * self.page_size

    def erase(self, batch_idx: torch.Tensor) -> None:
        """
        Removes a single batch from paged attention.

        Args:
            batch_idx (Tensor): batch index to be removed; shape :math:`(1)`.
        """

        # find allocated pages
        allocated_page_idx = self.page_table[batch_idx] != -1
        allocated_pages = self.page_table[batch_idx][allocated_page_idx]

        # clean metadata
        self.capacity[batch_idx] = 0
        self.empty_pages += allocated_pages.tolist()
        self.physical_to_logical[batch_idx][:, allocated_pages] = -1
        self.page_table[batch_idx] = -1
```
- **EN**: Declares `PagedAttention`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `PagedAttention`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 127-148
```python
    def assign(
        self,
        batch_idx: torch.Tensor,
        input_pos: torch.Tensor,
        k_val: torch.Tensor,
        v_val: torch.Tensor,
        k_cache: torch.Tensor,
        v_cache: torch.Tensor,
    ) -> None:
        """
        Assigns new contents `val` to the storage `cache` at the location
        `batch_idx` and `input_pos`.

        Args:
            batch_idx (Tensor): batch index; shape :math:`(B)`.
            input_pos (Tensor): input positions to be assigned for the given batch; shape :math:`(B, S)`.
            val (Tensor): value to be assigned; shape :math:`(B, H, S, D)`
            cache (Tensor): the cache to store the values; shape:`(1, H, MAX_S, D)`
        """
        if k_val.requires_grad:
            raise RuntimeError("val must not require gradient")
```
- **EN**: Defines the `PagedAttention.assign` method; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`PagedAttention.assign` 方法；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 149-166
```python
        B, H, S, K_D = k_val.shape
        V_D = v_val.shape[3]
        if B != batch_idx.shape[0]:
            raise RuntimeError(
                f"Expect val and batch_idx have the same batch size "
                f"but got B={B} and B={batch_idx.shape[0]}."
            )
        if H != k_cache.shape[1]:
            raise RuntimeError(
                f"Expect val and cache has the same number of heads "
                f"but got H={H} and H={k_cache.shape[1]}."
            )
        if S != input_pos.shape[1]:
            raise RuntimeError(
                f"Expect val and input_pos has the same length "
                f"but got S={S} and S={input_pos.shape[0]}."
            )
        if K_D != k_cache.shape[3]:
```
- **EN**: This block continues `PagedAttention.assign` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `PagedAttention.assign`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 167-193
```python
            raise RuntimeError(
                f"Expect k_val and k_cache has the same hidden dim "
                f"but got D={K_D} and D={k_cache.shape[3]}."
            )
        if V_D != v_cache.shape[3]:
            raise RuntimeError(
                f"Expect v_val and v_cache has the same hidden dim "
                f"but got D={V_D} and D={v_cache.shape[3]}."
            )

        # find address
        logical_block_idx = input_pos // self.page_size  # [B, S]
        logical_block_offset = input_pos % self.page_size  # [B, S]
        physical_block_idx = torch.gather(
            self.page_table[batch_idx], 1, logical_block_idx.to(torch.int64)
        ).to(torch.int32)  # [B, S]

        addr = (physical_block_idx * self.page_size + logical_block_offset).view(
            -1
        )  # [B*S]

        k_val = k_val.permute(1, 0, 2, 3).contiguous().view(1, H, B * S, K_D)
        v_val = v_val.permute(1, 0, 2, 3).contiguous().view(1, H, B * S, V_D)

        k_cache[:, :, addr, :] = k_val
        v_cache[:, :, addr, :] = v_val
```
- **EN**: This block continues `PagedAttention` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `PagedAttention`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 194-215
```python
    def convert_logical_block_mask(
        self,
        block_mask: BlockMask,
        batch_idx: torch.Tensor | None = None,
        kv_len: torch.Tensor | None = None,
    ) -> BlockMask:
        """
        Converts a logical block mask by mapping its logical kv indices to the corresponding
        physical kv indices.

        Args:
            block_mask (BlockMask): logical block mask;
                kv_indices shape :math:`(B, H, ROWS, MAX_BLOCKS_IN_COL)`.
            batch_idx (Tensor): batch index corresponding to the block_mask
                batch dimension. This provides flexibility to convert a
                block mask with smaller batch size than the page table;
                shape :math:`(B)`.
            kv_len (Optional[Tensor]): actual KV sequence length for upper bound check;
                shape :math:`(B,)` to handle multiple batches.
        """
        B, H, ROWS, MAX_BLOCKS_IN_COL = block_mask.kv_indices.shape
```
- **EN**: Defines the `PagedAttention.convert_logical_block_mask` method; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`PagedAttention.convert_logical_block_mask` 方法；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 216-235
```python
        if block_mask.BLOCK_SIZE[1] != self.page_size:
            raise RuntimeError(
                f"Expect block_mask has the same column block size as page_size"
                f"but got size={block_mask.BLOCK_SIZE[1]} and size={self.page_size}"
            )

        # Increase the num columns of converted block mask from logical block mask's
        # num columns to n_pages, since a) the converted block mask
        # may have larger indices values; and b) `_ordered_to_dense` realizes
        # a dense tensor with these converted indices. There would be an IndexError
        # if using the logical block mask's num columns.

        device = block_mask.kv_num_blocks.device

        if batch_idx is None:
            batch_idx = torch.arange(B, device=device)
        page_table = self.page_table[batch_idx]

        new_kv_num_blocks = block_mask.kv_num_blocks.clone()
```
- **EN**: This block continues `PagedAttention.convert_logical_block_mask` and works to normalize dtype/device related arguments and behavior. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `PagedAttention.convert_logical_block_mask`，用于规范化 dtype/device 相关参数与行为。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 236-253
```python
        new_kv_indices = torch.zeros(
            (B, H, ROWS, self.n_pages), dtype=torch.int32, device=device
        )
        new_kv_indices[:, :, :, :MAX_BLOCKS_IN_COL] = (
            torch.gather(
                page_table, 1, block_mask.kv_indices.view(B, -1).to(torch.int64)
            )
            .view(block_mask.kv_indices.shape)
            .to(torch.int32)
        )

        new_full_kv_indices, new_full_kv_num_blocks = None, None
        if block_mask.full_kv_num_blocks is not None:
            if block_mask.full_kv_indices is None:
                raise AssertionError(
                    "block_mask.full_kv_indices must not be None when full_kv_num_blocks is not None"
                )
            new_full_kv_num_blocks = block_mask.full_kv_num_blocks.clone()
```
- **EN**: This block continues `PagedAttention.convert_logical_block_mask` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `PagedAttention.convert_logical_block_mask`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 254-279
```python
            new_full_kv_indices = torch.zeros(
                (B, H, ROWS, self.n_pages), dtype=torch.int32, device=device
            )
            new_full_kv_indices[:, :, :, :MAX_BLOCKS_IN_COL] = (
                torch.gather(
                    page_table,
                    1,
                    block_mask.full_kv_indices.view(B, -1).to(torch.int64),
                )
                .view(block_mask.full_kv_indices.shape)
                .to(torch.int32)
            )

        new_mask_mod = self.get_mask_mod(block_mask.mask_mod, kv_len)

        seq_lengths = (block_mask.seq_lengths[0], self.n_pages * self.page_size)
        return BlockMask.from_kv_blocks(
            new_kv_num_blocks,
            new_kv_indices,
            new_full_kv_num_blocks,
            new_full_kv_indices,
            block_mask.BLOCK_SIZE,
            new_mask_mod,
            seq_lengths=seq_lengths,
        )
```
- **EN**: This block continues `PagedAttention` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `PagedAttention`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 280-297
```python
    def get_mask_mod(
        self,
        mask_mod: _mask_mod_signature | None,
        kv_len: torch.Tensor | None = None,
    ) -> _mask_mod_signature:
        """
        Converts a mask_mod based on mapping from the physical block index to the logical
        block index.

        Args:
            mask_mod (_mask_mod_signature): mask_mod based on the logical block index.
            kv_len (Optional[torch.Tensor]): actual KV sequence length for upper bound check.
        """
        if mask_mod is None:
            mask_mod = noop_mask

        def new_mask_mod(
            b: torch.Tensor,
```
- **EN**: Defines the `PagedAttention.get_mask_mod` method; this block introduces logic that prepare neural-network operators or module behavior.
- **CN**: 定义`PagedAttention.get_mask_mod` 方法；该代码块引入了用于准备神经网络算子或模块行为的逻辑。

### Lines 298-316
```python
            h: torch.Tensor,
            q_idx: torch.Tensor,
            physical_kv_idx: torch.Tensor,
        ):
            physical_kv_block = physical_kv_idx // self.page_size
            physical_kv_offset = physical_kv_idx % self.page_size
            logical_block_idx = self.physical_to_logical[b, physical_kv_block]
            logical_kv_idx = logical_block_idx * self.page_size + physical_kv_offset
            live_block = logical_block_idx >= 0
            within_upper_bound = (
                logical_kv_idx < kv_len[b] if kv_len is not None else True
            )
            within_lower_bound = logical_kv_idx >= 0
            is_valid = live_block & within_upper_bound & within_lower_bound

            return torch.where(is_valid, mask_mod(b, h, q_idx, logical_kv_idx), False)

        return new_mask_mod
```
- **EN**: This block continues `PagedAttention` and works to hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `PagedAttention`，用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果。

### Lines 317-334
```python
    def get_score_mod(
        self,
        score_mod: _score_mod_signature | None,
        kv_len: torch.Tensor | None = None,
    ) -> _score_mod_signature:
        """
        Converts a score_mod based on mapping from the physical block index to the logical
        block index.

        Args:
            score_mod (_score_mod_signature): score_mod based on the logical block index.
            `kv_len (Optional[torch.Tensor]): actual KV sequence length for upper bound check.

        """
        if score_mod is None:
            score_mod = _identity

        def new_score_mod(
```
- **EN**: Defines the `PagedAttention.get_score_mod` method; this block introduces logic that prepare neural-network operators or module behavior.
- **CN**: 定义`PagedAttention.get_score_mod` 方法；该代码块引入了用于准备神经网络算子或模块行为的逻辑。

### Lines 335-357
```python
            score: torch.Tensor,
            b: torch.Tensor,
            h: torch.Tensor,
            q_idx: torch.Tensor,
            physical_kv_idx: torch.Tensor,
        ):
            physical_kv_block = physical_kv_idx // self.page_size
            physical_kv_offset = physical_kv_idx % self.page_size
            logical_block_idx = self.physical_to_logical[b, physical_kv_block]
            logical_kv_idx = logical_block_idx * self.page_size + physical_kv_offset
            live_block = logical_block_idx >= 0
            within_upper_bound = (
                logical_kv_idx < kv_len[b] if kv_len is not None else True
            )
            within_lower_bound = logical_kv_idx >= 0
            is_valid = live_block & within_upper_bound & within_lower_bound

            return torch.where(
                is_valid,
                score_mod(score, b, h, q_idx, logical_kv_idx),
                float("-inf"),
            )
```
- **EN**: This block continues `PagedAttention.get_score_mod` and works to hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `PagedAttention.get_score_mod`，用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果。

### Lines 358-358
```python
        return new_score_mod
```
- **EN**: This block continues `PagedAttention.get_score_mod` and works to prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `PagedAttention.get_score_mod`，用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Attention mechanics / 注意力机制**
  - EN: The logic specializes in attention-style data movement, masking, or projection patterns.
  - CN: 该逻辑专门处理注意力风格的数据搬运、掩码或投影模式。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn.attention.flex_attention`
- **Primary symbols / 核心符号**: `__all__`, `_cdiv`, `PagedAttention`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
