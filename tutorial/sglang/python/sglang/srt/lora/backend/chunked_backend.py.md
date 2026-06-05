# chunked_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/backend/chunked_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a LoRA execution backend or backend interface for SGLang, defining how adapter weights are applied during model execution. It also exposes the data paths needed by CUDA-graph-friendly LoRA kernels. / 该文件实现了 SGLang 的 LoRA 执行后端或后端接口，定义了在模型执行期间如何应用适配器权重。它也提供了兼容 CUDA Graph 的 LoRA 内核所需的数据路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23: Module header, imports, and shared constants
```python
import dataclasses
from typing import List, Optional, Tuple

import torch

from sglang.srt.lora.backend.base_backend import BaseLoRABackend
from sglang.srt.lora.triton_ops import (
    chunked_embedding_lora_a_forward,
    chunked_sgmv_lora_expand_forward,
    chunked_sgmv_lora_shrink_forward,
)
from sglang.srt.lora.utils import (
    LoRABatchInfo,
    generate_sequence_lengths,
    get_lm_head_pruned_lens,
    merge_and_chunk_segments,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.server_args import ServerArgs

MIN_CHUNK_SIZE = 16
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 24-35: ChunkedSgmvLoRABackend class declaration
```python
class ChunkedSgmvLoRABackend(BaseLoRABackend):
    """
    Chunked LoRA backend using segmented matrix-vector multiplication.

    This backend is largely based on the SGMV (Segmented Gather Matrix-Vector multiplication) algorithm
    introduced in the Punica paper (https://arxiv.org/pdf/2310.18547). One main variation made here is to
    segment the input sequences into fixed-size chunks, which reduces excessive kernel launches especially
    when the LoRA distribution is skewed.
    """

    name = "csgmv"
```
**EN:** This block declares the `ChunkedSgmvLoRABackend` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ChunkedSgmvLoRABackend` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 36-43: ChunkedSgmvLoRABackend initializer
```python
    def __init__(
        self,
        max_loras_per_batch: int,
        device: torch.device,
        server_args: ServerArgs,
    ):
        super().__init__(max_loras_per_batch, device)
        self.max_chunk_size = server_args.max_lora_chunk_size
```
**EN:** This block initializes the `ChunkedSgmvLoRABackend` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `ChunkedSgmvLoRABackend` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 45-62: ChunkedSgmvLoRABackend.run_lora_a_embedding method
```python
    def run_lora_a_embedding(
        self,
        input_ids: torch.Tensor,
        weights: torch.Tensor,
        vocab_size: int,
        extra_embeddings: torch.Tensor = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        assert (
            extra_embeddings is None
        ), "Extra embeddings for lora a is not supported yet in chunked backend"
        return chunked_embedding_lora_a_forward(
            input_ids=input_ids,
            weights=weights,
            batch_info=self.batch_info,
            vocab_size=vocab_size,
        )
```
**EN:** This block uses `ChunkedSgmvLoRABackend.run_lora_a_embedding` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend.run_lora_a_embedding` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 64-81: ChunkedSgmvLoRABackend.run_lora_a_sgemm method
```python
    def run_lora_a_sgemm(
        self,
        x: torch.Tensor,
        weights: torch.Tensor,
        pruned_batch_info: LoRABatchInfo = None,
        stack_num: int = 1,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        batch_info = (
            pruned_batch_info if pruned_batch_info is not None else self.batch_info
        )
        return chunked_sgmv_lora_shrink_forward(
            x=x,
            weights=weights,
            batch_info=batch_info,
            num_slices=stack_num,
        )
```
**EN:** This block uses `ChunkedSgmvLoRABackend.run_lora_a_sgemm` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend.run_lora_a_sgemm` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 83-106: ChunkedSgmvLoRABackend.run_lora_b_sgemm method
```python
    def run_lora_b_sgemm(
        self,
        x: torch.Tensor,
        weights: torch.Tensor,
        output_offset: torch.Tensor,
        base_output: torch.Tensor = None,
        pruned_batch_info: LoRABatchInfo = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        # For simple lora B, we use slice offsets [0, output_dim]
        output_dim = weights.shape[-2]
        max_slice_size = output_dim
        batch_info = (
            pruned_batch_info if pruned_batch_info is not None else self.batch_info
        )
        return chunked_sgmv_lora_expand_forward(
            x=x,
            weights=weights,
            batch_info=batch_info,
            slice_offsets=output_offset,
            max_slice_size=max_slice_size,
            base_output=base_output,
        )
```
**EN:** This block uses `ChunkedSgmvLoRABackend.run_lora_b_sgemm` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend.run_lora_b_sgemm` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 108-140: ChunkedSgmvLoRABackend.run_qkv_lora method
```python
    def run_qkv_lora(
        self,
        x: torch.Tensor,
        qkv_lora_a: torch.Tensor,
        qkv_lora_b: torch.Tensor,
        output_offset: torch.Tensor,
        max_qkv_out_dim: int,
        base_output: torch.Tensor = None,
        n_slices: int = 3,
        *args,
        **kwargs,
    ) -> torch.Tensor:

        # x: (s, input_dim)
        # qkv_lora_a: (num_lora, n_slices * r, input_dim)
        # qkv_lora_b: (num_lora, total_output_dim, r)
        assert isinstance(qkv_lora_b, torch.Tensor)

        lora_a_output = chunked_sgmv_lora_shrink_forward(
            x=x,
            weights=qkv_lora_a,
            batch_info=self.batch_info,
            num_slices=n_slices,
        )
        lora_output = chunked_sgmv_lora_expand_forward(
            x=lora_a_output,
            weights=qkv_lora_b,
            batch_info=self.batch_info,
            slice_offsets=output_offset,
            max_slice_size=max_qkv_out_dim,
            base_output=base_output,
        )
        return lora_output
```
**EN:** This block uses `ChunkedSgmvLoRABackend.run_qkv_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend.run_qkv_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 142-174: ChunkedSgmvLoRABackend.run_gate_up_lora method
```python
    def run_gate_up_lora(
        self,
        x: torch.Tensor,
        gate_up_lora_a: torch.Tensor,
        gate_up_lora_b: torch.Tensor,
        output_offset: torch.Tensor,
        base_output: torch.Tensor = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:

        # x: (s, input_dim)
        # gate_up_lora_a: (num_lora, 2 * r, input_dim)
        # gate_up_lora_b: (num_lora, 2 * output_dim, r)
        assert isinstance(gate_up_lora_b, torch.Tensor)
        output_dim = gate_up_lora_b.shape[-2] // 2

        # lora_a_output: (s, 2 * r)
        lora_a_output = chunked_sgmv_lora_shrink_forward(
            x=x,
            weights=gate_up_lora_a,
            batch_info=self.batch_info,
            num_slices=2,
        )
        lora_output = chunked_sgmv_lora_expand_forward(
            x=lora_a_output,
            weights=gate_up_lora_b,
            batch_info=self.batch_info,
            slice_offsets=output_offset,
            max_slice_size=output_dim,
            base_output=base_output,
        )
        return lora_output
```
**EN:** This block uses `ChunkedSgmvLoRABackend.run_gate_up_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend.run_gate_up_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 176-191: ChunkedSgmvLoRABackend._determine_chunk_size method
```python
    def _determine_chunk_size(self, forward_batch: ForwardBatch) -> int:
        """
        Heuristically determine the chunk size based on token token number in a batch.

        Args:
            forward_batch (ForwardBatch): The batch information containing sequence lengths.

        Returns:
            The determined chunk size
        """
        num_tokens = (
            forward_batch.extend_num_tokens
            if forward_batch.forward_mode.is_extend()
            else forward_batch.batch_size
        )
        return self._determine_chunk_size_for_tokens(num_tokens)
```
**EN:** This block uses `ChunkedSgmvLoRABackend._determine_chunk_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend._determine_chunk_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 193-204: ChunkedSgmvLoRABackend._determine_chunk_size_for_tokens method
```python
    def _determine_chunk_size_for_tokens(self, num_tokens: int) -> int:
        """Determine chunk size given a token count directly."""
        if self.max_chunk_size <= MIN_CHUNK_SIZE:
            return MIN_CHUNK_SIZE

        if num_tokens >= 256:
            chunk_size = 128
        elif num_tokens >= 64:
            chunk_size = 32
        else:  # num_tokens < 64
            chunk_size = 16
        return min(self.max_chunk_size, chunk_size)
```
**EN:** This block uses `ChunkedSgmvLoRABackend._determine_chunk_size_for_tokens` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend._determine_chunk_size_for_tokens` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 206-216: ChunkedSgmvLoRABackend._build_req_seg_indptr method
```python
    @staticmethod
    def _build_req_seg_indptr(forward_batch: ForwardBatch) -> torch.Tensor:
        """Build per-request cumulative token boundaries on CPU (pinned)."""
        bs = forward_batch.batch_size
        if forward_batch.forward_mode.is_decode():
            indptr = torch.arange(bs + 1, dtype=torch.int32, pin_memory=True)
        else:
            seg_lens = generate_sequence_lengths(forward_batch, device="cpu")
            indptr = torch.zeros(bs + 1, dtype=torch.int32, pin_memory=True)
            torch.cumsum(seg_lens, dim=0, out=indptr[1:])
        return indptr
```
**EN:** This block uses `ChunkedSgmvLoRABackend._build_req_seg_indptr` to construct helper objects or layouts. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend._build_req_seg_indptr` 来构建辅助对象或布局。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 218-241: ChunkedSgmvLoRABackend.init_cuda_graph_batch_info method
```python
    def init_cuda_graph_batch_info(
        self,
        max_bs_in_cuda_graph: int,
        num_tokens_per_bs: int,
    ):
        max_num_segments = (
            (num_tokens_per_bs + MIN_CHUNK_SIZE - 1) // MIN_CHUNK_SIZE
        ) * max_bs_in_cuda_graph
        max_num_tokens = max_bs_in_cuda_graph * num_tokens_per_bs
        with torch.device("cuda"):
            self.cuda_graph_batch_info = LoRABatchInfo(
                bs=max_bs_in_cuda_graph,
                use_cuda_graph=True,
                seg_lens=torch.zeros(max_num_segments, dtype=torch.int32),
                seg_indptr=torch.zeros(max_num_segments + 1, dtype=torch.int32),
                weight_indices=torch.zeros(max_num_segments, dtype=torch.int32),
                permutation=torch.zeros(max_num_tokens, dtype=torch.int32),
                lora_ranks=torch.zeros(self.max_loras_per_batch, dtype=torch.int32),
                scalings=torch.zeros(self.max_loras_per_batch, dtype=torch.float),
                num_segments=None,  # Set per batch
                max_len=None,  # Not used in CSGMV backend
                req_seg_indptr=torch.zeros(max_bs_in_cuda_graph + 1, dtype=torch.int32),
                req_weight_indices=torch.zeros(max_bs_in_cuda_graph, dtype=torch.int32),
            )
```
**EN:** This block uses `ChunkedSgmvLoRABackend.init_cuda_graph_batch_info` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend.init_cuda_graph_batch_info` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 243-302: ChunkedSgmvLoRABackend.prepare_lora_batch method (part 1/2)
```python
    def prepare_lora_batch(
        self,
        forward_batch: ForwardBatch,
        weight_indices: list[int],
        lora_ranks: list[int],
        scalings: list[float],
        use_cuda_graph: bool,
    ):
        chunk_size = self._determine_chunk_size(forward_batch)

        permutation, weight_indices_reordered = ChunkedSgmvLoRABackend._get_permutation(
            seq_weight_indices=weight_indices,
            forward_batch=forward_batch,
        )

        seg_weight_indices, seg_indptr = self._get_segments_info(
            weights_reordered=weight_indices_reordered,
            chunk_size=chunk_size,
        )
        num_segments = len(seg_weight_indices)

        lora_ranks_tensor = torch.tensor(
            lora_ranks, dtype=torch.int32, pin_memory=True, device="cpu"
        )
        scalings_tensor = torch.tensor(
            scalings, dtype=torch.float, pin_memory=True, device="cpu"
        )

        bs = forward_batch.batch_size
        req_wi_tensor = torch.tensor(
            weight_indices, dtype=torch.int32, pin_memory=True, device="cpu"
        )
        req_seg_indptr_cpu = self._build_req_seg_indptr(forward_batch)

        if not use_cuda_graph:
            batch_info = LoRABatchInfo(
                bs=bs,
                num_segments=num_segments,
                max_len=chunk_size,
                use_cuda_graph=False,
                seg_indptr=torch.empty(
                    (num_segments + 1,), dtype=torch.int32, device=self.device
                ),
                weight_indices=torch.empty(
                    (num_segments,), dtype=torch.int32, device=self.device
                ),
                lora_ranks=torch.empty(
                    (self.max_loras_per_batch,), dtype=torch.int32, device=self.device
                ),
                scalings=torch.empty(
                    (self.max_loras_per_batch,), dtype=torch.float, device=self.device
                ),
                permutation=torch.empty(
                    (len(permutation),), dtype=torch.int32, device=self.device
                ),
                seg_lens=None,
                req_seg_indptr=torch.empty(
                    (bs + 1,), dtype=torch.int32, device=self.device
                ),
                req_weight_indices=torch.empty(
```
**EN:** This block uses `ChunkedSgmvLoRABackend.prepare_lora_batch` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend.prepare_lora_batch` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 303-330: ChunkedSgmvLoRABackend.prepare_lora_batch method (part 2/2)
```python
                    (bs,), dtype=torch.int32, device=self.device
                ),
            )
        else:
            batch_info = self.cuda_graph_batch_info
            batch_info.bs = bs
            batch_info.num_segments = num_segments
            batch_info.max_len = chunk_size

        # Copy to device asynchronously
        batch_info.lora_ranks[: self.max_loras_per_batch].copy_(
            lora_ranks_tensor, non_blocking=True
        )
        batch_info.scalings[: self.max_loras_per_batch].copy_(
            scalings_tensor, non_blocking=True
        )
        batch_info.weight_indices[:num_segments].copy_(
            seg_weight_indices, non_blocking=True
        )
        batch_info.seg_indptr[: num_segments + 1].copy_(seg_indptr, non_blocking=True)
        batch_info.permutation[: len(permutation)].copy_(permutation, non_blocking=True)
        batch_info.req_seg_indptr[: bs + 1].copy_(req_seg_indptr_cpu, non_blocking=True)
        batch_info.req_weight_indices[:bs].copy_(req_wi_tensor, non_blocking=True)

        self.batch_info = batch_info
        self.lm_head_batch_info, self.lm_head_pass_batch_infos = (
            self._prepare_lm_head_batch_info(forward_batch, weight_indices, batch_info)
        )
```
**EN:** This block uses `ChunkedSgmvLoRABackend.prepare_lora_batch` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend.prepare_lora_batch` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 332-373: ChunkedSgmvLoRABackend._prepare_lm_head_batch_info method
```python
    def _prepare_lm_head_batch_info(
        self,
        forward_batch: ForwardBatch,
        weight_indices: list[int],
        batch_info: LoRABatchInfo,
    ) -> Tuple[Optional[LoRABatchInfo], Optional[List[LoRABatchInfo]]]:

        # Precompute lm_head_batch_info for pruned lm_head LoRA
        pruned_lens = get_lm_head_pruned_lens(forward_batch)
        lm_head_batch_info = None
        lm_head_pass_batch_infos = None

        if pruned_lens is not None:
            pruned_total = sum(pruned_lens)
            chunk_size = self._determine_chunk_size_for_tokens(pruned_total)
            lm_head_segments = merge_and_chunk_segments(
                weight_indices, pruned_lens, chunk_size=chunk_size
            )
            lm_head_batch_info = self._build_lm_head_batch_info(
                lm_head_segments, batch_info, chunk_size, pruned_total
            )

            # Precompute per-pass batch_infos for logprobs chunking
            pass_segments = self._get_lm_head_pass_segments(weight_indices, pruned_lens)
            if pass_segments is not None:
                lm_head_pass_batch_infos = []
                for seg_wi, seg_lens_list in pass_segments:
                    pass_total = sum(seg_lens_list)
                    pass_chunk_size = self._determine_chunk_size_for_tokens(pass_total)
                    chunked_segments = merge_and_chunk_segments(
                        seg_wi, seg_lens_list, chunk_size=pass_chunk_size
                    )
                    lm_head_pass_batch_infos.append(
                        self._build_lm_head_batch_info(
                            chunked_segments,
                            batch_info,
                            pass_chunk_size,
                            pass_total,
                        )
                    )

        return lm_head_batch_info, lm_head_pass_batch_infos
```
**EN:** This block uses `ChunkedSgmvLoRABackend._prepare_lm_head_batch_info` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend._prepare_lm_head_batch_info` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 375-406: ChunkedSgmvLoRABackend._build_lm_head_batch_info method
```python
    def _build_lm_head_batch_info(
        self,
        lm_head_segments: Tuple[List[int], List[int]],
        batch_info: LoRABatchInfo,
        chunk_size: int,
        expected_tokens: int,
    ) -> LoRABatchInfo:
        seg_weight_indices_cpu, seg_lens_cpu = lm_head_segments
        pruned_total = sum(seg_lens_cpu)
        num_segments = len(seg_weight_indices_cpu)

        weight_indices = torch.tensor(
            seg_weight_indices_cpu, dtype=torch.int32, device=self.device
        )
        seg_lens = torch.tensor(seg_lens_cpu, dtype=torch.int32, device=self.device)
        seg_indptr = torch.zeros(
            (num_segments + 1,), dtype=torch.int32, device=self.device
        )
        seg_indptr[1:] = torch.cumsum(seg_lens, dim=0)

        # Identity permutation (lm_head tokens are in original order)
        permutation = torch.arange(pruned_total, dtype=torch.int32, device=self.device)

        return dataclasses.replace(
            batch_info,
            num_segments=num_segments,
            max_len=chunk_size,
            seg_indptr=seg_indptr,
            weight_indices=weight_indices,
            permutation=permutation,
            expected_tokens=expected_tokens,
        )
```
**EN:** This block uses `ChunkedSgmvLoRABackend._build_lm_head_batch_info` to construct helper objects or layouts. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend._build_lm_head_batch_info` 来构建辅助对象或布局。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 408-448: ChunkedSgmvLoRABackend._get_permutation method
```python
    @staticmethod
    def _get_permutation(seq_weight_indices, forward_batch: ForwardBatch):
        """
        Computes permutation indices for reordering tokens by their LoRA adapter assignments.

        This function implements the "gather" step in Chunked Segmented Gather Matrix Vector
        multiplication by creating a permutation that groups tokens by their LoRA adapter.
        Tokens using the same LoRA adapter are placed together to enable efficient batched
        computation.

        Example:
            seq_weight_indices = [0, 1, 0]  # 3 sequences using adapters [0, 1, 0]
            extend_seq_lens = [2, 1, 3]     # sequence lengths [2, 1, 3 tokens]

            # Creates row_weight_indices: [0, 0, 1, 0, 0, 0] (6 tokens total)
            # Returns permutation: [0, 1, 3, 4, 5, 2] (groups adapter 0 tokens together)
            # weights_reordered: [0, 0, 0, 0, 0, 1] (sorted by adapter)

        Args:
            seq_weight_indices: List of LoRA adapter indices for each sequence
            forward_batch (ForwardBatch): Batch information containing sequence lengths

        Returns:
            tuple: (permutation, weights_reordered) where:
                - permutation: Token reordering indices to group by adapter
                - weights_reordered: Sorted adapter indices for each token
        """
        with torch.device("cpu"):
            seq_weight_indices = torch.tensor(seq_weight_indices, dtype=torch.int32)
            seg_lens_cpu = generate_sequence_lengths(forward_batch)

            row_weight_indices = torch.repeat_interleave(
                seq_weight_indices, seg_lens_cpu
            )
            permutation = torch.empty(
                (len(row_weight_indices),), dtype=torch.long, pin_memory=True
            )
            torch.argsort(row_weight_indices, stable=True, out=permutation)
            weights_reordered = row_weight_indices[permutation]

            return permutation, weights_reordered
```
**EN:** This block uses `ChunkedSgmvLoRABackend._get_permutation` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend._get_permutation` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 450-509: ChunkedSgmvLoRABackend._get_segments_info method (part 1/2)
```python
    def _get_segments_info(self, weights_reordered: torch.Tensor, chunk_size: int):
        """
        Computes segment information for chunked SGMV operations.

        This function takes the reordered weight indices and creates segments of fixed size
        (self.segment_size) for efficient kernel execution. Each segment contains tokens
        that use the same LoRA adapter, enabling vectorized computation.

        The segmentation is necessary because:
        1. GPU kernels work efficiently on fixed-size blocks
        2. Large groups of tokens using the same adapter are split into manageable chunks
        3. Each segment can be processed independently in parallel

        Example:
            weights_reordered = [0, 0, 0, 0, 0, 1]  # 5 tokens with adapter 0, 1 with adapter 1
            segment_size = 3

            # Creates segments:
            # Segment 0: tokens 0-2 (adapter 0), length=3
            # Segment 1: tokens 3-4 (adapter 0), length=2
            # Segment 2: token 5 (adapter 1), length=1

            # Returns:
            # weight_indices_list: [0, 0, 1] (adapter for each segment)
            # seg_indptr: [0, 3, 5, 6] (cumulative segment boundaries)

        Args:
            weights_reordered (torch.Tensor): Sorted adapter indices for each token
            chunk_size (int): Fixed size for each segment

        Returns:
            tuple: (weight_indices_list, seg_indptr) where:
                - weight_indices_list: LoRA adapter index for each segment
                - seg_indptr: Cumulative segment boundaries (CSR-style indptr)
        """
        with torch.device("cpu"):
            unique_weights, counts = torch.unique_consecutive(
                weights_reordered, return_counts=True
            )

            weight_indices_list = []
            seg_lens_list = []

            for weight_idx, group_len in zip(unique_weights, counts):
                group_len = group_len.item()
                num_segs = (group_len + chunk_size - 1) // chunk_size

                weight_indices_list.extend([weight_idx.item()] * num_segs)
                seg_lens_list.extend([chunk_size] * (num_segs - 1))
                seg_lens_list.append(group_len - (num_segs - 1) * chunk_size)

            seg_lens = torch.tensor(seg_lens_list, dtype=torch.int32)

            weight_indices_list = torch.tensor(
                weight_indices_list, dtype=torch.int32, pin_memory=True
            )

            seg_indptr = torch.empty(
                (len(seg_lens) + 1,), dtype=torch.int32, pin_memory=True
            )
```
**EN:** This block uses `ChunkedSgmvLoRABackend._get_segments_info` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend._get_segments_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 510-513: ChunkedSgmvLoRABackend._get_segments_info method (part 2/2)
```python
            seg_indptr[0] = 0
            seg_indptr[1:] = torch.cumsum(seg_lens, dim=0)

            return weight_indices_list, seg_indptr
```
**EN:** This block uses `ChunkedSgmvLoRABackend._get_segments_info` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `ChunkedSgmvLoRABackend._get_segments_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Backend abstraction / 后端抽象
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.lora.backend.base_backend`
- `sglang.srt.lora.triton_ops`
- `sglang.srt.lora.utils`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.server_args`
### External / 外部
- `torch`
- `dataclasses` (stdlib)
- `typing` (stdlib)
