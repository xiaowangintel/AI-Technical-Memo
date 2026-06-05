# vit_npu_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/graph_runner/vit_npu_graph_runner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for backend-specific runtime support inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的后端运行时支持支持。

## Line-by-Line Analysis / 逐行分析
### Lines 15-30: Module setup and shared state / 模块设置与共享状态
```python
"""ViT NPU Graph Runner class."""

from __future__ import annotations

from typing import Dict, Hashable, List, Optional, Tuple

import torch
import torch.nn as nn
import torch_npu

from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    set_graph_pool_id,
)
from sglang.srt.layers.attention.vision import VisionAttention
from sglang.srt.multimodal.vit_cuda_graph_runner import ViTCudaGraphRunner
from sglang.srt.server_args import get_global_server_args
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `typing`, `torch`, `torch.nn`, `torch_npu`, `sglang.srt.distributed.device_communicators.pynccl_allocator`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `typing`, `torch`, `torch.nn`, `torch_npu`, `sglang.srt.distributed.device_communicators.pynccl_allocator`。

### Lines 33-44: Class `ViTNpuGraphRunner` declaration / 类 `ViTNpuGraphRunner` 声明
```python
class ViTNpuGraphRunner(ViTCudaGraphRunner):
    """Generic ViT NPU Graph Runner.

    This runner captures the "blocks + merger + deepstack merger (optional)" part
    of a vision transformer into a NPU graph and replays it for identical shapes.

    Optional for Qwen3 deepstack:
      - vit.deepstack_vision_indexes: Sequence[int]
      - vit.deepstack_merger_list: nn.ModuleList (same length as deepstack_vision_indexes)
    """

    _graph_memory_pool = None
```
**EN:** This class establishes `ViTNpuGraphRunner` as the main container/coordinator for the surrounding logic. It inherits from `ViTCudaGraphRunner`. Its core interface includes methods such as `__init__`, `device`, `dtype`, `_create_graph`, `create_graph`, `replay`.
**CN:** 该类将 `ViTNpuGraphRunner` 定义为周边逻辑的主要封装体或协调者。 它继承自 `ViTCudaGraphRunner`。 其核心接口包括 `__init__`, `device`, `dtype`, `_create_graph`, `create_graph`, `replay` 等方法。

### Lines 46-55: Method `ViTNpuGraphRunner.__init__` / 方法 `ViTNpuGraphRunner.__init__`
```python
    def __init__(
        self,
        vit: nn.Module,
    ) -> None:
        super().__init__(vit)
        self.device_module = torch.get_device_module(self.device)
        self.cu_seq_lens: Dict[Hashable, torch.Tensor] = {}

        # rotary position buffers shared across graphs
        self.sin_cos_ws: Dict[Hashable, Tuple[torch.Tensor, torch.Tensor]] = {}
```
**EN:** This method implements `__init__` on `ViTNpuGraphRunner`. It primarily calls `super.__init__`, `torch.get_device_module`, `super` to complete its work. State updates are written into `self.device_module`, `self.cu_seq_lens`, `self.sin_cos_ws`.
**CN:** 该方法（属于 `ViTNpuGraphRunner`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `torch.get_device_module`, `super` 来完成任务。 状态更新主要写入 `self.device_module`, `self.cu_seq_lens`, `self.sin_cos_ws`。

### Lines 57-59: Method `ViTNpuGraphRunner.device` / 方法 `ViTNpuGraphRunner.device`
```python
    @property
    def device(self) -> torch.device:
        return self.vit.device
```
**EN:** This method implements `device` on `ViTNpuGraphRunner`.
**CN:** 该方法（属于 `ViTNpuGraphRunner`）实现了 `device`。

### Lines 61-63: Method `ViTNpuGraphRunner.dtype` / 方法 `ViTNpuGraphRunner.dtype`
```python
    @property
    def dtype(self) -> torch.dtype:
        return self.vit.dtype
```
**EN:** This method implements `dtype` on `ViTNpuGraphRunner`.
**CN:** 该方法（属于 `ViTNpuGraphRunner`）实现了 `dtype`。

### Lines 65-126: Method `ViTNpuGraphRunner._create_graph` / 方法 `ViTNpuGraphRunner._create_graph`
```python
    def _create_graph(
        self,
        graph_key: int,
    ):

        graph = torch_npu.npu.NPUGraph()
        vit = self.vit

        override_backend = get_global_server_args().mm_attention_backend
        with torch_npu.npu.graph(graph, pool=ViTNpuGraphRunner._graph_memory_pool):
            y = None
            deepstack_outs: List[torch.Tensor] = []
            deepstack_capture_idx = 0

            for layer_num, blk in enumerate(vit.blocks):
                if override_backend == "ascend_attn":
                    cu_seq_lens = self.cu_seq_lens[graph_key]
                else:
                    raise RuntimeError("Not supported ViT attention backend")

                if layer_num == 0:
                    y = blk(
                        self.block_input[graph_key],
                        cu_seqlens=cu_seq_lens,
                        rotary_pos_emb_cos=self.sin_cos_ws[graph_key][0],
                        rotary_pos_emb_sin=self.sin_cos_ws[graph_key][1],
                        output_ws=self.block_ws[graph_key],
                    )
# ... omitted for brevity ...
                    [main_out] + deepstack_outs, dim=1
                )
            else:
                self.block_output[graph_key] = main_out

        self.block_graphs[graph_key] = graph
```
**EN:** This method implements `_create_graph` on `ViTNpuGraphRunner`. It primarily calls `torch_npu.npu.NPUGraph`, `get_global_server_args`, `torch_npu.npu.graph`, `enumerate`, `vit.merger`, `torch.cat` to complete its work. State updates are written into `graph`, `vit`, `override_backend`, `self.block_graphs`, `y`, `deepstack_outs`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该方法（属于 `ViTNpuGraphRunner`）实现了 `_create_graph`。 它主要通过调用 `torch_npu.npu.NPUGraph`, `get_global_server_args`, `torch_npu.npu.graph`, `enumerate`, `vit.merger`, `torch.cat` 来完成任务。 状态更新主要写入 `graph`, `vit`, `override_backend`, `self.block_graphs`, `y`, `deepstack_outs`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 128-175: Method `ViTNpuGraphRunner.create_graph` / 方法 `ViTNpuGraphRunner.create_graph`
```python
    def create_graph(
        self,
        x_3d: torch.Tensor,  # [S, 1, H]
        cu_seqlens: torch.Tensor,
        rotary_pos_emb_cos: Optional[torch.Tensor] = None,
        rotary_pos_emb_sin: Optional[torch.Tensor] = None,
    ) -> int:
        vit = self.vit
        graph_key = self._get_graph_key(x_3d)

        if graph_key in self.block_graphs:
            return graph_key

        if ViTNpuGraphRunner._graph_memory_pool is None:
            ViTNpuGraphRunner._graph_memory_pool = (
                self.device_module.graph_pool_handle()
            )
        # Set graph pool id globally to be able to use symmetric memory
        set_graph_pool_id(ViTNpuGraphRunner._graph_memory_pool)

        # pre-allocate workspace
        attn_module: VisionAttention = vit.blocks[0].attn
        num_heads = attn_module.num_attention_heads_per_partition
        attn_head_dim = attn_module.head_size

        if graph_key not in self.block_output:
            self.block_output[graph_key] = x_3d
            self.block_input[graph_key] = x_3d
# ... omitted for brevity ...
        if rotary_pos_emb_cos is not None and rotary_pos_emb_sin is not None:
            self._create_graph(
                graph_key=graph_key,
            )

        return graph_key
```
**EN:** This method implements `create_graph` on `ViTNpuGraphRunner`. It primarily calls `self._get_graph_key`, `set_graph_pool_id`, `self.device_module.graph_pool_handle`, `torch.empty`, `seq_lens.to.to`, `self._create_graph` to complete its work. State updates are written into `vit`, `graph_key`, `attn_module`, `num_heads`, `attn_head_dim`, `ViTNpuGraphRunner._graph_memory_pool`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ViTNpuGraphRunner`）实现了 `create_graph`。 它主要通过调用 `self._get_graph_key`, `set_graph_pool_id`, `self.device_module.graph_pool_handle`, `torch.empty`, `seq_lens.to.to`, `self._create_graph` 来完成任务。 状态更新主要写入 `vit`, `graph_key`, `attn_module`, `num_heads`, `attn_head_dim`, `ViTNpuGraphRunner._graph_memory_pool`。 实现中使用了条件分支。

### Lines 177-202: Method `ViTNpuGraphRunner.replay` / 方法 `ViTNpuGraphRunner.replay`
```python
    def replay(
        self,
        graph_key: int,
        x_3d: torch.Tensor,
        rotary_pos_emb_cos: Optional[torch.Tensor] = None,
        rotary_pos_emb_sin: Optional[torch.Tensor] = None,
        output_indices: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if rotary_pos_emb_cos is not None and rotary_pos_emb_sin is not None:
            # update rotary workspace content
            self.sin_cos_ws[graph_key][0].copy_(rotary_pos_emb_cos)
            self.sin_cos_ws[graph_key][1].copy_(rotary_pos_emb_sin)

        # copy input
        self.block_input[graph_key].copy_(x_3d)

        # replay
        self.block_graphs[graph_key].replay()

        out = self.block_output[graph_key]

        # Optional output reordering (Qwen2.5-VL window permutation inverse)
        if output_indices is not None:
            out = out.index_select(0, output_indices)

        return out
```
**EN:** This method implements `replay` on `ViTNpuGraphRunner`. It primarily calls `self.block_input.copy_`, `self.block_graphs.replay`, `self.sin_cos_ws.copy_`, `out.index_select` to complete its work. State updates are written into `out`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ViTNpuGraphRunner`）实现了 `replay`。 它主要通过调用 `self.block_input.copy_`, `self.block_graphs.replay`, `self.sin_cos_ws.copy_`, `out.index_select` 来完成任务。 状态更新主要写入 `out`。 实现中使用了条件分支。

### Lines 204-229: Method `ViTNpuGraphRunner.run` / 方法 `ViTNpuGraphRunner.run`
```python
    def run(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
        rotary_pos_emb_cos: Optional[torch.Tensor] = None,
        rotary_pos_emb_sin: Optional[torch.Tensor] = None,
        output_indices: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        # x: [seq_len, hidden] -> [S, B=1, H]
        x_3d = x.unsqueeze(1)
        graph_key = self._get_graph_key(x_3d)
        if graph_key not in self.block_graphs:
            self.create_graph(
                x_3d=x_3d,
                cu_seqlens=cu_seqlens,
                rotary_pos_emb_cos=rotary_pos_emb_cos,
                rotary_pos_emb_sin=rotary_pos_emb_sin,
            )

        return self.replay(
            graph_key=graph_key,
            x_3d=x_3d,
            rotary_pos_emb_cos=rotary_pos_emb_cos,
            rotary_pos_emb_sin=rotary_pos_emb_sin,
            output_indices=output_indices,
        )
```
**EN:** This method implements `run` on `ViTNpuGraphRunner`. It primarily calls `x.unsqueeze`, `self._get_graph_key`, `self.replay`, `self.create_graph` to complete its work. State updates are written into `x_3d`, `graph_key`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ViTNpuGraphRunner`）实现了 `run`。 它主要通过调用 `x.unsqueeze`, `self._get_graph_key`, `self.replay`, `self.create_graph` 来完成任务。 状态更新主要写入 `x_3d`, `graph_key`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `ViTNpuGraphRunner`
- **Functions / 函数**: `__init__`, `device`, `dtype`, `_create_graph`, `create_graph`, `replay`, `run`
- **Themes / 主题**: `runner`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.distributed.device_communicators.pynccl_allocator`, `sglang.srt.layers.attention.vision`, `sglang.srt.multimodal.vit_cuda_graph_runner`, `sglang.srt.server_args`
- **External / 外部依赖**: `torch`, `torch.nn`, `torch_npu`
- **Standard library / 标准库**: `__future__`, `typing`
