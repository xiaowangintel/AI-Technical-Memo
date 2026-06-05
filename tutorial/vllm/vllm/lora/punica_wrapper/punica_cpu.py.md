# punica_cpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/punica_wrapper/punica_cpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implement Punica wrapper operations with PyTorch-native CPU kernels for prefill and decode paths. / [CN] 使用 PyTorch 原生 CPU kernel 实现 Punica wrapper，在 prefill 与 decode 路径上执行 LoRA 运算。

## Line-by-Line Analysis / 逐行分析
### Prefill vs. decode kernels / Prefill 与 decode kernel 选择
```python
def _shrink_prefill(self, y, x, w_t_all, scale: float):
    if self.no_lora:
        return
    sgmv_shrink(
        x,
        w_t_all,
        y,
        *self.prefill_metadata,
        scale,
    )

def _shrink_decode(self, y, x, w_t_all, scale: float):
    bgmv_shrink(x, w_t_all, y, self.token_lora_indices, scale)

def _expand_prefill(self, y, x, w_t_all, add_inputs: bool):
    if self.no_lora:
        return
    sgmv_expand(
        x,
        w_t_all,
        y,
        *self.prefill_metadata,
        add_inputs,
    )

def _expand_decode(self, y, x, w_t_all, add_inputs: bool):
    bgmv_expand(x, w_t_all, y, self.token_lora_indices, add_inputs)
```
**EN:** CPU execution uses two operator families: `sgmv_*` for prefill, where grouped sequence metadata is available, and `bgmv_*` for decode, where token-wise mapping is cheaper. The `no_lora` guard skips useless prefill work.
**CN:** CPU 路径使用两套算子：prefill 阶段使用带分段元数据的 `sgmv_*`，decode 阶段使用按 token 映射更直接的 `bgmv_*`。`no_lora` 判断可以避免无意义的 prefill 计算。

### Unified helper dispatch / 统一辅助调度
```python
def _apply_expand(self, y, x, w_t_all, y_offset, y_slice_size, add_inputs=True):
    expand_slice_fun: Callable = (
        self._expand_slice_prefill if self.is_prefill else self._expand_slice_decode
    )
    expand_slice_fun(y, x, w_t_all, y_offset, y_slice_size, add_inputs)

def _apply_shrink(self, y, x, w_t_all, scale: float):
    y_org = y
    y = y.view(-1, y.shape[-1])
    shrink_fun: Callable = (
        self._shrink_prefill if self.is_prefill else self._shrink_decode
    )
    shrink_fun(y, x, w_t_all, scale)
    y = y.view_as(y_org)
```
**EN:** These helpers centralize the stage switch, so higher-level methods do not need separate prefill/decode branches. They flatten views before calling kernels to keep tensor shapes compatible.
**CN:** 这些辅助函数把阶段切换统一封装起来，因此上层接口不必分别写 prefill/decode 分支。调用 kernel 前会先展平视图，以匹配底层算子的期望形状。

### Multi-slice shrink and expand / 多切片 shrink 与 expand
```python
def add_shrink(self, y, x, lora_a_stacked, scale: float, **kwargs):
    x = x.view(-1, x.shape[-1])
    for slice_idx in range(len(lora_a_stacked)):
        self._apply_shrink(y[slice_idx], x, lora_a_stacked[slice_idx], scale)

def add_expand(
    self,
    y: torch.Tensor,
    x: tuple[torch.Tensor, ...] | torch.Tensor,
    lora_b_stacked: tuple[torch.Tensor, ...],
    output_slices: tuple[int, ...],
    offset_start: int = 0,
    add_inputs=True,
    **kwargs,
) -> None:
    y_org = y
    y = y.view(-1, y.shape[-1])
    offset_left = offset_start
    for slice_idx in range(len(lora_b_stacked)):
        self._apply_expand(
            y,
            x[slice_idx],
            lora_b_stacked[slice_idx],
            offset_left,
            output_slices[slice_idx],
            add_inputs=add_inputs,
        )
        offset_left += output_slices[slice_idx]
    y = y.view_as(y_org)
```
**EN:** `add_shrink` iterates over LoRA-A slices, while `add_expand` writes each LoRA-B slice into a different output range. This matches models that split projection matrices into multiple logical parts.
**CN:** `add_shrink` 逐个处理 LoRA-A 切片，`add_expand` 则把每个 LoRA-B 切片写入不同的输出区间。这与很多模型把投影矩阵拆成多个逻辑部分的结构相匹配。

### Composed linear path / 组合式 linear 路径
```python
def add_lora_linear(
    self,
    y: torch.Tensor,
    x: torch.Tensor,
    lora_a_stacked: tuple[torch.Tensor, ...],
    lora_b_stacked: tuple[torch.Tensor, ...],
    scale: float,
    output_slices: tuple[int, ...],
    *,
    buffer: tuple[torch.Tensor, ...] | None = None,
    **kwargs,
) -> None:
    assert len(lora_a_stacked) == len(lora_b_stacked) == len(output_slices)
    if buffer is None:
        r = lora_b_stacked[0].size(-1)
        buffer = tuple(
            torch.zeros((x.size(0), r), dtype=torch.float32, device=x.device)
            for _ in range(len(output_slices))
        )
    self.add_shrink(buffer, x, lora_a_stacked, scale, **kwargs)
    self.add_expand(
        y, buffer, lora_b_stacked, output_slices, add_inputs=True, **kwargs
    )
```
**EN:** Linear LoRA is implemented as a two-step pipeline: project into rank space with LoRA-A, then expand back with LoRA-B. The intermediate buffer defaults to `float32`, matching higher-precision behavior used by the Triton backend.
**CN:** 线性层的 LoRA 采用两步流水：先通过 LoRA-A 投影到低秩空间，再通过 LoRA-B 展开回输出空间。中间缓冲区默认使用 `float32`，与 Triton 后端的高精度策略保持一致。

### Embedding and logits paths / Embedding 与 logits 路径
```python
def add_lora_embedding(self, y, x, lora_b_stacked, add_inputs: bool = True, **kwargs):
    expand_fun: Callable = (
        self._expand_prefill if self.is_prefill else self._expand_decode
    )
    expand_fun(y, x, lora_b_stacked, add_inputs)

def add_lora_logits(self, y, x, lora_a_stacked, lora_b_stacked, scale, *, buffer=None, **kwargs):
    y_org = y
    y = y.view(-1, y.shape[-1])
    x = x.view(-1, x.shape[-1])
    r = lora_b_stacked.size(-1)
    if buffer is None:
        buffer = torch.zeros((x.size(0), r), dtype=torch.float32, device=x.device)
    bgmv_shrink(x, lora_a_stacked, buffer, self.sampler_indices, scale)
    bgmv_expand(buffer, lora_b_stacked, y, self.sampler_indices, add_inputs=True)
    y = y.view_as(y_org)
```
**EN:** Embedding layers only need the expand half because the input is already an embedding lookup result. Logits always use sampler-level indices and the `bgmv_*` operators, reflecting per-request rather than per-token adapter selection.
**CN:** Embedding 层只需要 expand 半段，因为输入已经是 embedding 查找结果。logits 路径始终使用 sampler 级索引和 `bgmv_*` 算子，体现的是按请求而非按 token 选择 adapter。

## Key Concepts / 关键概念
- `sgmv_*` for grouped prefill, `bgmv_*` for decode. / prefill 用 `sgmv_*`，decode 用 `bgmv_*`。
- Shared helper functions hide stage-specific branching. / 共享辅助函数屏蔽阶段分支。
- Linear LoRA = shrink then expand. / 线性 LoRA = 先 shrink 再 expand。

## Dependencies / 依赖关系
- `vllm.lora.ops.torch_ops`: `bgmv_*`, `sgmv_*`
- `PunicaWrapperBase`
- Metadata properties from `punica_base.py`
