# tbo_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/tbo_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the tbo backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 tbo backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-7: imports
```python
from typing import TYPE_CHECKING, Callable, List, Optional

import torch

from sglang.srt.batch_overlap import two_batch_overlap
from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.speculative.spec_info import SpecInput
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 9-10: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 13-13: class TboAttnBackend
```python
class TboAttnBackend(AttentionBackend):
```
**EN:** Concrete attention backend that connects tbo attn backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 tbo attn backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 14-17: method TboAttnBackend.__init__
```python
    def __init__(self, primary: AttentionBackend, children: List[AttentionBackend]):
        super().__init__()
        self.primary = primary
        self.children = children
```
**EN:** Initializes the TboAttnBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TboAttnBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 19-24: method TboAttnBackend.init_new
```python
    @classmethod
    def init_new(cls, creator: Callable[[], AttentionBackend]):
        return cls(
            primary=creator(),
            children=[creator() for _ in range(2)],
        )
```
**EN:** Prepares init new so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init new，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 26-33: method TboAttnBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: "ForwardBatch"):
        self.primary.init_forward_metadata(forward_batch=forward_batch)
        if forward_batch.tbo_children is not None:
            for child, forward_batch_child in zip(
                self.children, forward_batch.tbo_children, strict=True
            ):
                if forward_batch_child.batch_size > 0:
                    child.init_forward_metadata(forward_batch=forward_batch_child)
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 35-39: method TboAttnBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        self.primary.init_cuda_graph_state(max_bs=max_bs, max_num_tokens=max_num_tokens)
        for item in self.children:
            # TODO for children, maybe can provide *smaller* max_bs to optimize
            item.init_cuda_graph_state(max_bs=max_bs, max_num_tokens=max_num_tokens)
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 41-70: method TboAttnBackend.init_forward_metadata_capture_cuda_graph
```python
    def init_forward_metadata_capture_cuda_graph(
        self,
        bs: int,
        num_tokens: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: "ForwardMode",
        spec_info: Optional[SpecInput],
    ):
        self.primary.init_forward_metadata_capture_cuda_graph(
            bs=bs,
            num_tokens=num_tokens,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            encoder_lens=encoder_lens,
            forward_mode=forward_mode,
            spec_info=spec_info,
        )

        self._init_forward_metadata_cuda_graph_children(
            fn_name="init_forward_metadata_capture_cuda_graph",
            bs=bs,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            encoder_lens=encoder_lens,
            forward_mode=forward_mode,
            spec_info=spec_info,
            capture_num_tokens=num_tokens,
        )
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 72-104: method TboAttnBackend.init_forward_metadata_replay_cuda_graph
```python
    def init_forward_metadata_replay_cuda_graph(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_sum: int,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: "ForwardMode",
        spec_info: Optional[SpecInput],
        seq_lens_cpu: Optional[torch.Tensor],
    ):
        self.primary.init_forward_metadata_replay_cuda_graph(
            bs=bs,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            seq_lens_sum=seq_lens_sum,
            encoder_lens=encoder_lens,
            forward_mode=forward_mode,
            spec_info=spec_info,
            seq_lens_cpu=seq_lens_cpu,
        )

        self._init_forward_metadata_cuda_graph_children(
            fn_name="init_forward_metadata_replay_cuda_graph",
            bs=bs,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            encoder_lens=encoder_lens,
            forward_mode=forward_mode,
            spec_info=spec_info,
            replay_seq_lens_sum=seq_lens_sum,
            replay_seq_lens_cpu=seq_lens_cpu,
        )
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 106-174: method TboAttnBackend._init_forward_metadata_cuda_graph_children
```python
    def _init_forward_metadata_cuda_graph_children(
        self,
        fn_name: str,
        # common args
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: "ForwardMode",
        spec_info: Optional[SpecInput],
        # capture args
        capture_num_tokens: int = None,
        # replay args
        replay_seq_lens_sum: int = None,
        replay_seq_lens_cpu: Optional[torch.Tensor] = None,
    ):
        token_num_per_seq = two_batch_overlap.get_token_num_per_seq(
            forward_mode=forward_mode, spec_info=spec_info
        )
        if fn_name == "init_forward_metadata_capture_cuda_graph":
            assert (
                capture_num_tokens == bs * token_num_per_seq
            ), "For target-verify or decode mode, num_tokens should be equal to token_num_per_seq * bs"
        num_tokens = bs * token_num_per_seq

        tbo_split_seq_index, tbo_split_token_index = (
            two_batch_overlap.compute_split_indices_for_cuda_graph_replay(
                forward_mode=forward_mode,
                cuda_graph_num_tokens=num_tokens,
                spec_info=spec_info,
            )
        )
# ... omitted 25 lines ...
            seq_slice=slice(None, tbo_split_seq_index),
            **common_pre_split_args,
        )
        args_right = _init_forward_metadata_cuda_graph_split(
            output_bs=bs_child_right,
            seq_slice=slice(tbo_split_seq_index, None),
            **common_pre_split_args,
        )

        child_left, child_right = self.children
        getattr(child_left, fn_name)(**args_left)
        getattr(child_right, fn_name)(**args_right)
```
**EN:** Runs the forward-path logic for init forward metadata cuda graph children, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata cuda graph children 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 176-180: method TboAttnBackend.get_cuda_graph_seq_len_fill_value
```python
    def get_cuda_graph_seq_len_fill_value(self):
        ans = self.primary.get_cuda_graph_seq_len_fill_value()
        for child in self.children:
            assert ans == child.get_cuda_graph_seq_len_fill_value()
        return ans
```
**EN:** Computes and returns get cuda graph seq len fill value from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cuda graph seq len fill value。

### Lines 182-183: method TboAttnBackend.forward
```python
    def forward(self, *args, **kwargs):
        return self.primary.forward(*args, **kwargs)
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 185-186: method TboAttnBackend.forward_extend
```python
    def forward_extend(self, *args, **kwargs):
        return self.primary.forward_extend(*args, **kwargs)
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 188-189: method TboAttnBackend.forward_decode
```python
    def forward_decode(self, *args, **kwargs):
        return self.primary.forward_decode(*args, **kwargs)
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 191-192: method TboAttnBackend.get_indexer_metadata
```python
    def get_indexer_metadata(self, layer_id: int, forward_batch: "ForwardBatch"):
        return self.primary.get_indexer_metadata(layer_id, forward_batch)
```
**EN:** Computes and returns get indexer metadata from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer metadata。

### Lines 195-266: function _init_forward_metadata_cuda_graph_split
```python
def _init_forward_metadata_cuda_graph_split(
    fn_name: str,
    seq_slice: slice,
    output_bs: int,
    # common args
    bs: int,
    req_pool_indices: torch.Tensor,
    seq_lens: torch.Tensor,
    encoder_lens: Optional[torch.Tensor],
    forward_mode: "ForwardMode",
    spec_info: Optional[SpecInput],
    # capture args
    capture_num_tokens: int = None,
    # replay args
    replay_seq_lens_sum: int = None,
    replay_seq_lens_cpu: Optional[torch.Tensor] = None,
):
    token_num_per_seq = two_batch_overlap.get_token_num_per_seq(
        forward_mode=forward_mode, spec_info=spec_info
    )
    assert encoder_lens is None, "encoder_lens is not supported yet"
    if spec_info is not None:
        output_spec_info = two_batch_overlap.split_spec_info(
            spec_info=spec_info,
            start_seq_index=seq_slice.start if seq_slice.start is not None else 0,
            end_seq_index=seq_slice.stop if seq_slice.stop is not None else bs,
            start_token_index=(
                seq_slice.start * token_num_per_seq
                if seq_slice.start is not None
                else 0
            ),
            end_token_index=(
# ... omitted 28 lines ...
    elif fn_name == "init_forward_metadata_replay_cuda_graph":
        output_seq_lens_cpu = replay_seq_lens_cpu[seq_slice]
        ans.update(
            dict(
                seq_lens_sum=output_seq_lens_cpu.sum().item(),
                seq_lens_cpu=output_seq_lens_cpu,
            )
        )
    else:
        raise NotImplementedError

    return ans
```
**EN:** Runs the forward-path logic for init forward metadata cuda graph split, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata cuda graph split 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `typing.TYPE_CHECKING`
- `typing.Callable`
- `typing.List`
- `typing.Optional`
- `torch`
- `sglang.srt.batch_overlap.two_batch_overlap`
- `sglang.srt.layers.attention.base_attn_backend.AttentionBackend`
- `sglang.srt.speculative.spec_info.SpecInput`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
