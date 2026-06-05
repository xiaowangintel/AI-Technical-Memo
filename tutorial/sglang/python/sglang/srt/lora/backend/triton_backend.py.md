# triton_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/backend/triton_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a LoRA execution backend or backend interface for SGLang, defining how adapter weights are applied during model execution. It also exposes the data paths needed by CUDA-graph-friendly LoRA kernels. / 该文件实现了 SGLang 的 LoRA 执行后端或后端接口，定义了在模型执行期间如何应用适配器权重。它也提供了兼容 CUDA Graph 的 LoRA 内核所需的数据路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21: Module header, imports, and shared constants
```python
import dataclasses
from typing import List, Optional, Tuple

import torch

from sglang.srt.lora.backend.base_backend import BaseLoRABackend
from sglang.srt.lora.triton_ops import (
    embedding_lora_a_fwd,
    gate_up_lora_b_fwd,
    qkv_lora_b_fwd,
    sgemm_lora_a_fwd,
    sgemm_lora_b_fwd,
)
from sglang.srt.lora.utils import (
    LoRABatchInfo,
    get_lm_head_pruned_lens,
    merge_and_chunk_segments,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 22-24: TritonLoRABackend class declaration
```python
class TritonLoRABackend(BaseLoRABackend):
    name = "triton"
```
**EN:** This block declares the `TritonLoRABackend` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `TritonLoRABackend` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 25-31: TritonLoRABackend initializer
```python
    def __init__(
        self,
        max_loras_per_batch: int,
        device: torch.device,
        **kwargs,
    ):
        super().__init__(max_loras_per_batch, device)
```
**EN:** This block initializes the `TritonLoRABackend` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `TritonLoRABackend` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 33-49: TritonLoRABackend.run_lora_a_embedding method
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
        """Run LoRA A embedding lookup using Triton kernel."""
        return embedding_lora_a_fwd(
            input_ids=input_ids,
            weights=weights,
            batch_info=self.batch_info,
            vocab_size=vocab_size,
            extra_embeddings=extra_embeddings,
        )
```
**EN:** This block uses `TritonLoRABackend.run_lora_a_embedding` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend.run_lora_a_embedding` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 51-55: TritonLoRABackend._sgemm_info method
```python
    def _sgemm_info(self, pruned_batch_info=None):
        """Return the sgemm batch_info (merged segments when available)."""
        if pruned_batch_info is not None:
            return pruned_batch_info
        return getattr(self, "sgemm_batch_info", None) or self.batch_info
```
**EN:** This block uses `TritonLoRABackend._sgemm_info` to store configuration or metadata. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend._sgemm_info` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 57-68: TritonLoRABackend.run_lora_a_sgemm method
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
        return sgemm_lora_a_fwd(
            x, weights, self._sgemm_info(pruned_batch_info), stack_num=stack_num
        )
```
**EN:** This block uses `TritonLoRABackend.run_lora_a_sgemm` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend.run_lora_a_sgemm` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 70-81: TritonLoRABackend.run_lora_b_sgemm method
```python
    def run_lora_b_sgemm(
        self,
        x: torch.Tensor,
        weights: torch.Tensor,
        base_output: torch.Tensor = None,
        pruned_batch_info: LoRABatchInfo = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        return sgemm_lora_b_fwd(
            x, weights, self._sgemm_info(pruned_batch_info), base_output
        )
```
**EN:** This block uses `TritonLoRABackend.run_lora_b_sgemm` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend.run_lora_b_sgemm` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 83-112: TritonLoRABackend.run_qkv_lora method
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

        sgemm_info = self._sgemm_info()
        lora_a_output = sgemm_lora_a_fwd(x, qkv_lora_a, sgemm_info, stack_num=n_slices)
        lora_output = qkv_lora_b_fwd(
            lora_a_output,
            qkv_lora_b,
            sgemm_info,
            output_offset,
            max_qkv_out_dim,
            base_output,
            n_slices=n_slices,
        )
        return lora_output
```
**EN:** This block uses `TritonLoRABackend.run_qkv_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend.run_qkv_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 114-140: TritonLoRABackend.run_gate_up_lora method
```python
    def run_gate_up_lora(
        self,
        x: torch.Tensor,
        gate_up_lora_a: torch.Tensor,
        gate_up_lora_b: torch.Tensor,
        base_output: torch.Tensor = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:

        # x: (s, input_dim)
        # gate_up_lora_a: (num_lora, 2 * r, input_dim)
        # gate_up_lora_b: (num_lora, 2 * output_dim, r)
        assert isinstance(gate_up_lora_b, torch.Tensor)
        output_dim = gate_up_lora_b.shape[-2] // 2

        sgemm_info = self._sgemm_info()
        # lora_a_output: (s, 2 * r)
        lora_a_output = sgemm_lora_a_fwd(x, gate_up_lora_a, sgemm_info, stack_num=2)
        lora_output = gate_up_lora_b_fwd(
            lora_a_output,
            gate_up_lora_b,
            sgemm_info,
            output_dim,
            base_output,
        )
        return lora_output
```
**EN:** This block uses `TritonLoRABackend.run_gate_up_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend.run_gate_up_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 142-184: TritonLoRABackend.init_cuda_graph_batch_info method
```python
    def init_cuda_graph_batch_info(
        self,
        max_bs_in_cuda_graph: int,
        num_tokens_per_bs: int,
    ):
        max_tokens = max_bs_in_cuda_graph * num_tokens_per_bs
        mlpb = self.max_loras_per_batch
        with torch.device("cuda"):
            self.cuda_graph_batch_info = LoRABatchInfo(
                bs=max_bs_in_cuda_graph,
                use_cuda_graph=True,
                num_segments=None,
                seg_lens=torch.full(
                    (max_bs_in_cuda_graph,), num_tokens_per_bs, dtype=torch.int32
                ),
                seg_indptr=torch.zeros(max_bs_in_cuda_graph + 1, dtype=torch.int32),
                max_len=num_tokens_per_bs,
                weight_indices=torch.zeros(max_bs_in_cuda_graph, dtype=torch.int32),
                lora_ranks=torch.zeros(mlpb, dtype=torch.int32),
                scalings=torch.zeros(mlpb, dtype=torch.float),
                permutation=None,
            )

            torch.cumsum(
                self.cuda_graph_batch_info.seg_lens[:max_bs_in_cuda_graph],
                dim=0,
                out=self.cuda_graph_batch_info.seg_indptr[1 : max_bs_in_cuda_graph + 1],
            )

            # Sgemm batch_info with segments merged by adapter.
            # Updated each batch by compute_sgemm_routing().
            self.cuda_graph_sgemm_batch_info = LoRABatchInfo(
                bs=mlpb,
                use_cuda_graph=True,
                num_segments=mlpb,
                seg_lens=torch.zeros(mlpb, dtype=torch.int32),
                seg_indptr=torch.zeros(mlpb + 1, dtype=torch.int32),
                max_len=max_tokens,
                weight_indices=torch.arange(mlpb, dtype=torch.int32),
                lora_ranks=torch.zeros(mlpb, dtype=torch.int32),
                scalings=torch.zeros(mlpb, dtype=torch.float),
                permutation=torch.zeros(max_tokens, dtype=torch.int32),
            )
```
**EN:** This block uses `TritonLoRABackend.init_cuda_graph_batch_info` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend.init_cuda_graph_batch_info` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 186-227: TritonLoRABackend.compute_sgemm_routing method
```python
    def compute_sgemm_routing(self, use_cuda_graph: bool):
        """Sort tokens by adapter and build merged segments for sgemm LoRA."""
        bi = self.batch_info
        bs = bi.bs
        mlpb = self.max_loras_per_batch
        wi = bi.weight_indices[:bs]

        perm = torch.argsort(wi, stable=True).to(torch.int32)
        sorted_wi = wi[perm]
        adapter_ids = torch.arange(mlpb, device=wi.device, dtype=torch.int32)
        seg_starts = torch.searchsorted(sorted_wi, adapter_ids)
        seg_ends = torch.searchsorted(sorted_wi, adapter_ids, right=True)
        seg_lens = seg_ends - seg_starts

        if use_cuda_graph:
            sgemm = getattr(self, "cuda_graph_sgemm_batch_info", None)
            if sgemm is None:
                return
            sgemm.permutation[:bs] = perm
            sgemm.seg_lens[:] = seg_lens
            sgemm.seg_indptr[0:1].zero_()
            torch.cumsum(sgemm.seg_lens, dim=0, out=sgemm.seg_indptr[1:])
            sgemm.max_len = bs
            sgemm.lora_ranks[:mlpb] = bi.lora_ranks[:mlpb]
            sgemm.scalings[:mlpb] = bi.scalings[:mlpb]
        else:
            seg_indptr = torch.zeros(mlpb + 1, dtype=torch.int32, device=wi.device)
            seg_indptr[1:] = torch.cumsum(seg_lens, dim=0)
            sgemm = LoRABatchInfo(
                bs=mlpb,
                use_cuda_graph=False,
                num_segments=mlpb,
                seg_lens=seg_lens,
                seg_indptr=seg_indptr,
                max_len=bs,
                weight_indices=adapter_ids,
                lora_ranks=bi.lora_ranks[:mlpb].clone(),
                scalings=bi.scalings[:mlpb].clone(),
                permutation=perm,
            )

        self.sgemm_batch_info = sgemm
```
**EN:** This block uses `TritonLoRABackend.compute_sgemm_routing` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend.compute_sgemm_routing` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 229-288: TritonLoRABackend.prepare_lora_batch method (part 1/2)
```python
    def prepare_lora_batch(
        self,
        forward_batch: ForwardBatch,
        weight_indices: list[int],
        lora_ranks: list[int],
        scalings: list[float],
        use_cuda_graph: bool,
    ):
        # Use pinned memory to avoid synchronizations during host-to-device transfer
        weight_indices_tensor = torch.tensor(
            weight_indices, dtype=torch.int32, pin_memory=True, device="cpu"
        )
        lora_ranks_tensor = torch.tensor(
            lora_ranks, dtype=torch.int32, pin_memory=True, device="cpu"
        )
        scalings_tensor = torch.tensor(
            scalings, dtype=torch.float, pin_memory=True, device="cpu"
        )

        bs = forward_batch.batch_size

        if use_cuda_graph:
            assert (
                self.cuda_graph_batch_info is not None
            ), "CUDA Graph batch info is not initialized."
            batch_info = self.cuda_graph_batch_info
            batch_info.bs = forward_batch.batch_size
            batch_info.num_segments = forward_batch.batch_size
        else:
            max_len = (
                # Calculate max_len from the CPU copy to avoid D2H transfer.
                max(forward_batch.extend_seq_lens_cpu)
                if forward_batch.forward_mode.is_extend()
                else 1
            )
            seg_lens = (
                forward_batch.extend_seq_lens
                if forward_batch.forward_mode.is_extend()
                else torch.ones(bs, dtype=torch.int32, device=self.device)
            )
            seg_indptr = torch.zeros((bs + 1,), dtype=torch.int32, device=self.device)
            seg_indptr[1:] = torch.cumsum(seg_lens, dim=0)

            batch_info = LoRABatchInfo(
                bs=forward_batch.batch_size,
                num_segments=forward_batch.batch_size,
                max_len=max_len,
                use_cuda_graph=False,
                seg_lens=seg_lens,
                seg_indptr=seg_indptr,
                weight_indices=torch.empty(
                    (bs,), dtype=torch.int32, device=self.device
                ),
                lora_ranks=torch.empty(
                    (self.max_loras_per_batch,), dtype=torch.int64, device=self.device
                ),
                scalings=torch.empty(
                    (self.max_loras_per_batch,), dtype=torch.float, device=self.device
                ),
                permutation=None,
```
**EN:** This block uses `TritonLoRABackend.prepare_lora_batch` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `TritonLoRABackend.prepare_lora_batch` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 289-311: TritonLoRABackend.prepare_lora_batch method (part 2/2)
```python
            )

        # Copy to device asynchronously
        batch_info.lora_ranks[: self.max_loras_per_batch].copy_(
            lora_ranks_tensor, non_blocking=True
        )
        batch_info.scalings[: self.max_loras_per_batch].copy_(
            scalings_tensor, non_blocking=True
        )
        batch_info.weight_indices[:bs].copy_(weight_indices_tensor, non_blocking=True)

        self.batch_info = batch_info

        # Biggest win is in decode.
        is_decode = not forward_batch.forward_mode.is_extend()
        if is_decode:
            self.compute_sgemm_routing(use_cuda_graph)
        else:
            self.sgemm_batch_info = None

        self.lm_head_batch_info, self.lm_head_pass_batch_infos = (
            self._prepare_lm_head_batch_info(forward_batch, weight_indices, batch_info)
        )
```
**EN:** This block uses `TritonLoRABackend.prepare_lora_batch` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `TritonLoRABackend.prepare_lora_batch` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 313-349: TritonLoRABackend._prepare_lm_head_batch_info method
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
            lm_head_segments = merge_and_chunk_segments(
                weight_indices, pruned_lens, chunk_size=pruned_total
            )
            lm_head_batch_info = self._build_lm_head_batch_info(
                lm_head_segments, batch_info, pruned_total
            )

            # Precompute per-pass batch_infos for logprobs chunking
            pass_segments = self._get_lm_head_pass_segments(weight_indices, pruned_lens)
            if pass_segments is not None:
                lm_head_pass_batch_infos = []
                for seg_wi, seg_lens_list in pass_segments:
                    pass_total = sum(seg_lens_list)
                    merged_segments = merge_and_chunk_segments(
                        seg_wi, seg_lens_list, chunk_size=pass_total
                    )
                    self.lm_head_pass_batch_infos.append(
                        self._build_lm_head_batch_info(
                            merged_segments, batch_info, pass_total
                        )
                    )

        return lm_head_batch_info, lm_head_pass_batch_infos
```
**EN:** This block uses `TritonLoRABackend._prepare_lm_head_batch_info` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend._prepare_lm_head_batch_info` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 351-377: TritonLoRABackend._build_lm_head_batch_info method
```python
    def _build_lm_head_batch_info(
        self,
        lm_head_segments: Tuple[List[int], List[int]],
        batch_info: LoRABatchInfo,
        expected_tokens: int,
    ) -> LoRABatchInfo:
        seg_weight_indices_cpu, seg_lens_cpu = lm_head_segments
        num_segments = len(seg_weight_indices_cpu)

        seg_lens = torch.tensor(seg_lens_cpu, dtype=torch.int32, device=self.device)
        seg_indptr = torch.zeros(
            (num_segments + 1,), dtype=torch.int32, device=self.device
        )
        seg_indptr[1:] = torch.cumsum(seg_lens, dim=0)

        return dataclasses.replace(
            batch_info,
            bs=num_segments,
            num_segments=num_segments,
            max_len=max(seg_lens_cpu),
            seg_lens=seg_lens,
            seg_indptr=seg_indptr,
            weight_indices=torch.tensor(
                seg_weight_indices_cpu, dtype=torch.int32, device=self.device
            ),
            expected_tokens=expected_tokens,
        )
```
**EN:** This block uses `TritonLoRABackend._build_lm_head_batch_info` to construct helper objects or layouts. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TritonLoRABackend._build_lm_head_batch_info` 来构建辅助对象或布局。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

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
### External / 外部
- `torch`
- `dataclasses` (stdlib)
- `typing` (stdlib)
