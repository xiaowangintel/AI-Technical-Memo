# ascend_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/backend/ascend_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements a LoRA execution backend or backend interface for SGLang, defining how adapter weights are applied during model execution. It also exposes the data paths needed by CUDA-graph-friendly LoRA kernels. / 该文件实现了 SGLang 的 LoRA 执行后端或后端接口，定义了在模型执行期间如何应用适配器权重。它也提供了兼容 CUDA Graph 的 LoRA 内核所需的数据路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Module header, imports, and shared constants
```python
import torch

from sglang.srt.lora.backend.base_backend import BaseLoRABackend
from sglang.srt.lora.utils import LoRABatchInfo
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.utils import is_npu

if is_npu():
    import sgl_kernel_npu  # noqa: F401
    import torch_npu  # noqa: F401
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 13-15: AscendLoRABackend class declaration
```python
class AscendLoRABackend(BaseLoRABackend):
    name = "ascend"
```
**EN:** This block declares the `AscendLoRABackend` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `AscendLoRABackend` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 16-22: AscendLoRABackend initializer
```python
    def __init__(
        self,
        max_loras_per_batch: int,
        device: torch.device,
        **kwargs,
    ):
        super().__init__(max_loras_per_batch, device)
```
**EN:** This block initializes the `AscendLoRABackend` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `AscendLoRABackend` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 24-49: AscendLoRABackend.run_lora_a_sgemm method
```python
    def run_lora_a_sgemm(
        self, x: torch.Tensor, weights: torch.Tensor, *args, **kwargs
    ) -> torch.Tensor:

        total_seq_len, _ = x.shape
        _, weight_out_dim, _ = weights.shape

        output_tensor = torch.zeros(
            (total_seq_len, weight_out_dim), dtype=x.dtype, device=x.device
        )
        torch.ops.npu.sgmv_shrink(
            x,
            weights,
            self.batch_info.weight_indices,
            self.batch_info.seg_lens,
            output_tensor,
            1.0,
        )
        scaling = (
            self.batch_info.scalings.gather(0, self.batch_info.weight_indices)
            .repeat_interleave(self.batch_info.seg_lens, output_size=total_seq_len)
            .unsqueeze(-1)
        )
        output_tensor *= scaling

        return output_tensor
```
**EN:** This block uses `AscendLoRABackend.run_lora_a_sgemm` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AscendLoRABackend.run_lora_a_sgemm` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 51-79: AscendLoRABackend.run_lora_b_sgemm method
```python
    def run_lora_b_sgemm(
        self,
        x: torch.Tensor,
        weights: torch.Tensor,
        base_output: torch.Tensor = None,
        *args,
        **kwargs,
    ) -> torch.Tensor:
        total_seq_len, _ = x.shape
        _, weight_out_dim, _ = weights.shape

        if base_output is None:
            output_tensor = torch.zeros(
                (total_seq_len, weight_out_dim), device=x.device, dtype=x.dtype
            )
        else:
            output_tensor = base_output

        torch.ops.npu.sgmv_expand(
            x,
            weights,
            self.batch_info.weight_indices,
            self.batch_info.seg_lens,
            output_tensor,
            0,
            weight_out_dim,
        )

        return output_tensor
```
**EN:** This block uses `AscendLoRABackend.run_lora_b_sgemm` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AscendLoRABackend.run_lora_b_sgemm` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 81-140: AscendLoRABackend.run_qkv_lora method (part 1/2)
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
        assert isinstance(qkv_lora_b, torch.Tensor)

        total_seq_len, _ = x.shape
        _, weight_intermediate_dim, _ = qkv_lora_a.shape
        _, weight_out_dim, _ = qkv_lora_b.shape
        max_rank = weight_intermediate_dim // n_slices

        if base_output is None:
            output_tensor = torch.zeros(
                (total_seq_len, weight_out_dim), device=x.device, dtype=x.dtype
            )
        else:
            output_tensor = base_output

        lora_a_output = torch.zeros(
            total_seq_len, weight_intermediate_dim, dtype=x.dtype, device=x.device
        )
        torch.ops.npu.sgmv_shrink(
            x,
            qkv_lora_a,
            self.batch_info.weight_indices,
            self.batch_info.seg_lens,
            lora_a_output,
            1.0,
        )

        scaling = (
            self.batch_info.scalings.gather(0, self.batch_info.weight_indices)
            .repeat_interleave(self.batch_info.seg_lens, output_size=total_seq_len)
            .unsqueeze(-1)
        )
        lora_a_output *= scaling

        for slice_id in range(n_slices):
            slice_offset = output_offset_cpu[slice_id]
            slice_offset_next = output_offset_cpu[slice_id + 1]
            slice_size = slice_offset_next - slice_offset
            torch.ops.npu.sgmv_expand(
                lora_a_output[:, (max_rank * slice_id) : (max_rank * (slice_id + 1))],
                qkv_lora_b[:, slice_offset:slice_offset_next],
                self.batch_info.weight_indices,
                self.batch_info.seg_lens,
                output_tensor,
                slice_offset,
                slice_size,
            )
```
**EN:** This block uses `AscendLoRABackend.run_qkv_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `AscendLoRABackend.run_qkv_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 141-141: AscendLoRABackend.run_qkv_lora method (part 2/2)
```python
        return output_tensor
```
**EN:** This block uses `AscendLoRABackend.run_qkv_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `AscendLoRABackend.run_qkv_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 143-202: AscendLoRABackend.run_gate_up_lora method
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

        num_slices = 2
        assert isinstance(gate_up_lora_b, torch.Tensor)

        total_seq_len, _ = x.shape
        _, weight_intermediate_dim, _ = gate_up_lora_a.shape
        _, weight_out_dim, _ = gate_up_lora_b.shape
        slice_size = weight_out_dim // num_slices
        max_rank = weight_intermediate_dim // num_slices

        if base_output is None:
            output_tensor = torch.zeros(
                (total_seq_len, weight_out_dim), device=x.device, dtype=x.dtype
            )
        else:
            output_tensor = base_output

        lora_a_output = torch.zeros(
            total_seq_len, weight_intermediate_dim, dtype=x.dtype, device=x.device
        )

        torch.ops.npu.sgmv_shrink(
            x,
            gate_up_lora_a,
            self.batch_info.weight_indices,
            self.batch_info.seg_lens,
            lora_a_output,
            1.0,
        )

        scaling = (
            self.batch_info.scalings.gather(0, self.batch_info.weight_indices)
            .repeat_interleave(self.batch_info.seg_lens, output_size=total_seq_len)
            .unsqueeze(-1)
        )
        lora_a_output *= scaling

        slice_offset = 0
        for slice_id in range(num_slices):
            torch.ops.npu.sgmv_expand(
                lora_a_output[:, (max_rank * slice_id) : (max_rank * (slice_id + 1))],
                gate_up_lora_b[:, slice_offset : slice_offset + slice_size],
                self.batch_info.weight_indices,
                self.batch_info.seg_lens,
                output_tensor,
                slice_offset,
                slice_size,
            )
            slice_offset += slice_size

        return output_tensor
```
**EN:** This block uses `AscendLoRABackend.run_gate_up_lora` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AscendLoRABackend.run_gate_up_lora` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 204-231: AscendLoRABackend.init_cuda_graph_batch_info method
```python
    def init_cuda_graph_batch_info(
        self,
        max_bs_in_cuda_graph: int,
        num_tokens_per_bs: int,
    ):
        with torch.device("npu"):
            self.npu_graph_batch_info = LoRABatchInfo(
                bs=max_bs_in_cuda_graph,
                use_cuda_graph=True,
                num_segments=None,
                seg_lens=torch.full(
                    (max_bs_in_cuda_graph,), num_tokens_per_bs, dtype=torch.int32
                ),
                seg_indptr=torch.empty(max_bs_in_cuda_graph + 1, dtype=torch.int32),
                max_len=num_tokens_per_bs,
                weight_indices=torch.zeros(max_bs_in_cuda_graph, dtype=torch.int32),
                lora_ranks=torch.zeros(self.max_loras_per_batch, dtype=torch.int32),
                scalings=torch.zeros(self.max_loras_per_batch, dtype=torch.float),
                permutation=None,
            )

            # Initialize seg_indptr for NPU graph as they remain constant
            # across batches.
            torch.cumsum(
                self.npu_graph_batch_info.seg_lens[:max_bs_in_cuda_graph],
                dim=0,
                out=self.npu_graph_batch_info.seg_indptr[1 : max_bs_in_cuda_graph + 1],
            )
```
**EN:** This block uses `AscendLoRABackend.init_cuda_graph_batch_info` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AscendLoRABackend.init_cuda_graph_batch_info` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 233-292: AscendLoRABackend.prepare_lora_batch method (part 1/2)
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
                self.npu_graph_batch_info is not None
            ), "NPU Graph batch info is not initialized."
            batch_info = self.npu_graph_batch_info
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
                    (self.max_loras_per_batch,), dtype=torch.int32, device=self.device
                ),
                scalings=torch.empty(
                    (self.max_loras_per_batch,), dtype=torch.float, device=self.device
                ),
                permutation=None,
```
**EN:** This block uses `AscendLoRABackend.prepare_lora_batch` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `AscendLoRABackend.prepare_lora_batch` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 293-303: AscendLoRABackend.prepare_lora_batch method (part 2/2)
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
```
**EN:** This block uses `AscendLoRABackend.prepare_lora_batch` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `AscendLoRABackend.prepare_lora_batch` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Backend abstraction / 后端抽象
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.lora.backend.base_backend`
- `sglang.srt.lora.utils`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.utils`
### External / 外部
- `sgl_kernel_npu`
- `torch`
- `torch_npu`
