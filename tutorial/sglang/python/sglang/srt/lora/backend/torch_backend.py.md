# torch_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/backend/torch_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a LoRA execution backend or backend interface for SGLang, defining how adapter weights are applied during model execution. It also exposes the data paths needed by CUDA-graph-friendly LoRA kernels. / 该文件实现了 SGLang 的 LoRA 执行后端或后端接口，定义了在模型执行期间如何应用适配器权重。它也提供了兼容 CUDA Graph 的 LoRA 内核所需的数据路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15: Module header, imports, and shared constants
```python
from dataclasses import dataclass
from typing import Optional

import torch

from sglang.srt.lora.backend.base_backend import BaseLoRABackend
from sglang.srt.lora.torch_ops import (
    sgemm_lora_a_embedding_fwd,
    sgemm_lora_a_fwd,
    sgemm_lora_b_fwd,
)
from sglang.srt.lora.utils import LoRABatchInfo, generate_sequence_lengths
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 17-31: TorchNativeLoRABatchInfo class declaration
```python
class TorchNativeLoRABatchInfo(LoRABatchInfo):
    # ranks of each lora adapter, in shape (lora_num,) placed on cpu device
    lora_ranks_cpu: Optional[torch.Tensor] = None

    # Indice pointers of each segment in shape (num_segments + 1, ) placed on cpu device
    seg_indptr_cpu: Optional[torch.Tensor] = None

    # Lengths of each segments in shape (num_segments,) placed on cpu device
    seg_lens_cpu: Optional[torch.Tensor] = None

    # The index of lora adapter used by each segment, in shape (num_segments,) placed on cpu device
    weight_indices_cpu: Optional[torch.Tensor] = None

    # Scaling factors for each lora adapter, in shape (lora_num,) placed on cpu device
    scalings_cpu: Optional[torch.Tensor] = None
```
**EN:** This block declares the `TorchNativeLoRABatchInfo` class, which exists to store configuration or metadata. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `TorchNativeLoRABatchInfo` 类，其职责是存储配置或元数据。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 34-36: TorchNativeLoRABackend class declaration
```python
class TorchNativeLoRABackend(BaseLoRABackend):
    name = "torch_native"
```
**EN:** This block declares the `TorchNativeLoRABackend` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `TorchNativeLoRABackend` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 37-43: TorchNativeLoRABackend initializer
```python
    def __init__(
        self,
        max_loras_per_batch: int,
        device: torch.device,
        **kwargs,
    ):
        super().__init__(max_loras_per_batch, device)
```
**EN:** This block initializes the `TorchNativeLoRABackend` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `TorchNativeLoRABackend` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 45-64: TorchNativeLoRABackend.run_lora_a_embedding method
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
        output_tensor = sgemm_lora_a_embedding_fwd(
            inputs=input_ids,
            weights=weights,
            batch_info=self.batch_info,
            vocab_size=vocab_size,
        )

        return output_tensor
```
**EN:** This block uses `TorchNativeLoRABackend.run_lora_a_embedding` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TorchNativeLoRABackend.run_lora_a_embedding` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 66-81: TorchNativeLoRABackend.run_lora_a_sgemm method
```python
    def run_lora_a_sgemm(
        self,
        x: torch.Tensor,
        weights: torch.Tensor,
        stack_num: int = 1,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        output_tensor = sgemm_lora_a_fwd(
            inputs=x,
            weights=weights,
            batch_info=self.batch_info,
            num_slices=stack_num,
        )

        return output_tensor
```
**EN:** This block uses `TorchNativeLoRABackend.run_lora_a_sgemm` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TorchNativeLoRABackend.run_lora_a_sgemm` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 83-102: TorchNativeLoRABackend.run_lora_b_sgemm method
```python
    def run_lora_b_sgemm(
        self,
        x: torch.Tensor,
        weights: torch.Tensor,
        output_offset_cpu: torch.Tensor,
        base_output: torch.Tensor = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        _, weight_out_dim, _ = weights.shape

        output_tensor = sgemm_lora_b_fwd(
            inputs=x,
            weights=weights,
            batch_info=self.batch_info,
            slice_offsets=output_offset_cpu,
            base_output=base_output,
        )

        return output_tensor
```
**EN:** This block uses `TorchNativeLoRABackend.run_lora_b_sgemm` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TorchNativeLoRABackend.run_lora_b_sgemm` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 104-132: TorchNativeLoRABackend.run_qkv_lora method
```python
    def run_qkv_lora(
        self,
        x: torch.Tensor,
        qkv_lora_a: torch.Tensor,
        qkv_lora_b: torch.Tensor,
        output_offset: torch.Tensor,
        output_offset_cpu: torch.Tensor,
        max_qkv_out_dim: int,
        base_output: torch.Tensor = None,
        n_slices: int = 3,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        lora_a_output = sgemm_lora_a_fwd(
            inputs=x,
            weights=qkv_lora_a,
            batch_info=self.batch_info,
            num_slices=n_slices,
        )

        output_tensor = sgemm_lora_b_fwd(
            inputs=lora_a_output,
            weights=qkv_lora_b,
            batch_info=self.batch_info,
            slice_offsets=output_offset_cpu,
            base_output=base_output,
        )

        return output_tensor
```
**EN:** This block uses `TorchNativeLoRABackend.run_qkv_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TorchNativeLoRABackend.run_qkv_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 134-162: TorchNativeLoRABackend.run_gate_up_lora method
```python
    def run_gate_up_lora(
        self,
        x: torch.Tensor,
        gate_up_lora_a: torch.Tensor,
        gate_up_lora_b: torch.Tensor,
        output_offset_cpu: torch.Tensor,
        base_output: torch.Tensor = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        num_slices = len(output_offset_cpu) - 1
        _, weight_out_dim, _ = gate_up_lora_b.shape

        lora_a_output = sgemm_lora_a_fwd(
            inputs=x,
            weights=gate_up_lora_a,
            batch_info=self.batch_info,
            num_slices=num_slices,
        )

        output_tensor = sgemm_lora_b_fwd(
            inputs=lora_a_output,
            weights=gate_up_lora_b,
            batch_info=self.batch_info,
            slice_offsets=output_offset_cpu,
            base_output=base_output,
        )

        return output_tensor
```
**EN:** This block uses `TorchNativeLoRABackend.run_gate_up_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TorchNativeLoRABackend.run_gate_up_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 164-191: TorchNativeLoRABackend.init_cuda_graph_batch_info method
```python
    def init_cuda_graph_batch_info(
        self,
        max_bs_in_cuda_graph: int,
        num_tokens_per_bs: int,
    ):
        with torch.device("cuda"):
            self.cuda_graph_batch_info = TorchNativeLoRABatchInfo(
                use_cuda_graph=True,
                bs=max_bs_in_cuda_graph,
                num_segments=self.max_loras_per_batch,
                seg_lens=torch.full(
                    (max_bs_in_cuda_graph,), num_tokens_per_bs, dtype=torch.int32
                ),
                seg_indptr=torch.zeros(max_bs_in_cuda_graph + 1, dtype=torch.int32),
                weight_indices=torch.zeros(max_bs_in_cuda_graph, dtype=torch.int32),
                lora_ranks=torch.zeros(self.max_loras_per_batch, dtype=torch.int32),
                scalings=torch.zeros(self.max_loras_per_batch, dtype=torch.float),
                permutation=None,
                max_len=num_tokens_per_bs,
            )

            # Initialize seg_indptr for CUDA graph as they remain constant
            # across batches.
            torch.cumsum(
                self.cuda_graph_batch_info.seg_lens[:max_bs_in_cuda_graph],
                dim=0,
                out=self.cuda_graph_batch_info.seg_indptr[1 : max_bs_in_cuda_graph + 1],
            )
```
**EN:** This block uses `TorchNativeLoRABackend.init_cuda_graph_batch_info` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TorchNativeLoRABackend.init_cuda_graph_batch_info` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 193-252: TorchNativeLoRABackend.prepare_lora_batch method (part 1/2)
```python
    def prepare_lora_batch(
        self,
        forward_batch: ForwardBatch,
        weight_indices: list[int],
        lora_ranks: list[int],
        scalings: list[float],
        use_cuda_graph: bool,
    ):
        # Do not use merge optimization for graph mode
        # Use pinned memory to avoid synchronizations during host-to-device transfer
        original_seq_lens_cpu = generate_sequence_lengths(forward_batch, device="cpu")
        if not use_cuda_graph:
            original_weight_indices_tensor = torch.tensor(
                weight_indices, dtype=torch.int32, device="cpu"
            )

            unique_weight_indices_tensor, inverse_weight_indices_tensor = (
                torch.unique_consecutive(
                    original_weight_indices_tensor, return_inverse=True
                )
            )

            seg_lens_cpu = (
                torch.zeros_like(
                    unique_weight_indices_tensor, dtype=torch.int32, device="cpu"
                )
                .scatter_add_(
                    0,
                    inverse_weight_indices_tensor,
                    original_seq_lens_cpu,
                )
                .pin_memory()
            )

            weight_indices_tensor = unique_weight_indices_tensor.pin_memory()
        else:
            weight_indices_tensor = torch.repeat_interleave(
                torch.tensor(weight_indices, dtype=torch.int32, device="cpu"),
                original_seq_lens_cpu,
            ).pin_memory()
            seg_lens_cpu = torch.ones_like(weight_indices_tensor).pin_memory()

        seg_indptr_cpu = torch.zeros(
            (len(seg_lens_cpu) + 1,), dtype=torch.int32, pin_memory=True
        )
        seg_indptr_cpu[1:] = torch.cumsum(seg_lens_cpu, dim=0)
        lora_ranks_tensor = torch.tensor(
            lora_ranks, dtype=torch.int32, pin_memory=True, device="cpu"
        )
        scalings_tensor = torch.tensor(
            scalings, dtype=torch.float, pin_memory=True, device="cpu"
        )

        bs = forward_batch.batch_size
        num_segments = len(weight_indices_tensor)

        if use_cuda_graph:
            assert (
                self.cuda_graph_batch_info is not None
            ), "CUDA Graph batch info is not initialized."
```
**EN:** This block uses `TorchNativeLoRABackend.prepare_lora_batch` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `TorchNativeLoRABackend.prepare_lora_batch` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 253-301: TorchNativeLoRABackend.prepare_lora_batch method (part 2/2)
```python
            batch_info = self.cuda_graph_batch_info
            batch_info.bs = forward_batch.batch_size
            batch_info.num_segments = num_segments
        else:
            max_len = max(seg_lens_cpu)

            batch_info = TorchNativeLoRABatchInfo(
                bs=forward_batch.batch_size,
                num_segments=num_segments,
                max_len=max_len,
                use_cuda_graph=False,
                seg_lens=torch.empty((bs,), dtype=torch.int32, device=self.device),
                seg_indptr=torch.empty(
                    (bs + 1,), dtype=torch.int32, device=self.device
                ),
                weight_indices=torch.empty(
                    (bs,), dtype=torch.int32, device=self.device
                ),
                lora_ranks=torch.empty(
                    (self.max_loras_per_batch,), dtype=torch.int32, device=self.device
                ),
                scalings=torch.empty(
                    (self.max_loras_per_batch,), dtype=torch.float, device=self.device
                ),
                permutation=None,
            )

        # Copy to device asynchronously
        batch_info.lora_ranks[: self.max_loras_per_batch].copy_(
            lora_ranks_tensor, non_blocking=True
        )
        batch_info.scalings[: self.max_loras_per_batch].copy_(
            scalings_tensor, non_blocking=True
        )
        batch_info.weight_indices[:num_segments].copy_(
            weight_indices_tensor, non_blocking=True
        )
        batch_info.seg_indptr[: len(seg_indptr_cpu)].copy_(
            seg_indptr_cpu, non_blocking=True
        )
        batch_info.seg_lens[: len(seg_lens_cpu)].copy_(seg_lens_cpu, non_blocking=True)

        batch_info.lora_ranks_cpu = lora_ranks_tensor
        batch_info.seg_indptr_cpu = seg_indptr_cpu
        batch_info.seg_lens_cpu = seg_lens_cpu
        batch_info.weight_indices_cpu = weight_indices_tensor
        batch_info.scalings_cpu = scalings_tensor

        self.batch_info = batch_info
```
**EN:** This block uses `TorchNativeLoRABackend.prepare_lora_batch` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `TorchNativeLoRABackend.prepare_lora_batch` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Backend abstraction / 后端抽象
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.lora.backend.base_backend`
- `sglang.srt.lora.torch_ops`
- `sglang.srt.lora.utils`
- `sglang.srt.model_executor.forward_batch_info`
### External / 外部
- `torch`
- `dataclasses` (stdlib)
- `typing` (stdlib)
