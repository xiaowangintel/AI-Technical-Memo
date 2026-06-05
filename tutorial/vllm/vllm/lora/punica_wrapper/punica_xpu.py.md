# punica_xpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/punica_wrapper/punica_xpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implement the XPU/Intel Punica backend using XPU kernels for standard LoRA paths and Triton-based fused MoE helpers. / [CN] 实现 XPU/Intel 版 Punica 后端，标准 LoRA 路径使用 XPU kernel，融合式 MoE 路径复用 Triton 辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Initialization and dynamic shapes / 初始化与动态形状
```python
def __init__(
    self,
    max_num_batched_tokens: int,
    max_batches: int,
    device: torch.device | str,
    **kwargs,
):
    PunicaWrapperBase.__init__(self, max_num_batched_tokens, max_batches, device)
    torch._dynamo.mark_dynamic(self._token_lora_indices, 0)
    torch._dynamo.mark_dynamic(self._embeddings_indices, 1)
    torch._dynamo.mark_dynamic(self._sampler_indices_padded, 0)

    self.lora_config = kwargs["lora_config"]
    self.max_loras = self.lora_config.max_loras
    captured_lora_counts = get_captured_lora_counts(
        self.max_loras, self.lora_config.specialize_active_lora
    )
    self.token_mapping_meta = LoRAKernelMeta.make(
        self.max_loras,
        max_num_batched_tokens,
        device=device,
        captured_lora_counts=captured_lora_counts,
    )
    self.prompt_mapping_meta = LoRAKernelMeta.make(
        self.max_loras,
        max_num_batched_tokens,
        device=device,
        captured_lora_counts=captured_lora_counts,
    )
```
**EN:** XPU keeps the same high-level metadata design as GPU, but first marks several buffers as dynamic for `torch._dynamo` so compiled graphs can tolerate variable batch lengths.
**CN:** XPU 在高层元数据设计上与 GPU 基本一致，但会先用 `torch._dynamo` 把多个缓冲区标记为动态维度，以便编译图适配不同 batch 长度。

### Metadata refresh and narrowed index view / 元数据刷新与裁剪索引视图
```python
def update_metadata(self, mapping: LoRAMapping, lora_index_to_id, max_loras, vocab_size, **kwargs):
    self.is_prefill = mapping.is_prefill
    self._update_base_metadata(mapping, lora_index_to_id, max_loras, vocab_size)
    self.token_mapping_meta.prepare_tensors(self.token_lora_indices)
    self.prompt_mapping_meta.prepare_tensors(self.sampler_indices)

def _get_token_lora_indices(self, x: torch.Tensor) -> torch.IntTensor:
    return torch.narrow(self._token_lora_indices, 0, 0, x.size(0))
```
**EN:** Like the GPU backend, XPU refreshes compact metadata tensors after every mapping change. `_get_token_lora_indices` avoids copying by returning a view of exactly the active token range.
**CN:** 与 GPU 后端一样，XPU 在每次映射变化后都会刷新紧凑元数据张量。`_get_token_lora_indices` 通过返回当前有效 token 范围的视图来避免额外拷贝。

### Standard LoRA ops / 标准 LoRA 运算
```python
def _apply_shrink(self, y, x, w_t_all, scale: float):
    bgmv_shrink(x, w_t_all, y, self._get_token_lora_indices(x), scale)

def _apply_expand(self, y, x, w_t_all, y_offset, y_slice_size, add_inputs: bool):
    token_lora_indices = self._get_token_lora_indices(x)
    bgmv_expand_slice(
        x, w_t_all, y, token_lora_indices, y_offset, y_slice_size, add_inputs
    )

def add_shrink(self, y, x, lora_a_stacked, scale: float, **kwargs):
    x = x.view(-1, x.shape[-1])
    for slice_idx in range(len(lora_a_stacked)):
        self._apply_shrink(y[slice_idx], x, lora_a_stacked[slice_idx], scale)
```
**EN:** Basic XPU LoRA paths use `bgmv_*` kernels and iterate over slices explicitly, unlike the GPU backend that can batch more work into Triton kernels. The narrowed token-index view is reused for each call.
**CN:** 基础 XPU LoRA 路径使用 `bgmv_*` kernel，并显式遍历各个切片；这与 GPU 后端可将更多工作打包进 Triton kernel 的方式不同。每次调用都会复用裁剪后的 token 索引视图。

### Linear, embedding, and logits / Linear、embedding 与 logits
```python
def add_lora_embedding(self, y, x, lora_b_stacked, add_inputs: bool = True, **kwargs):
    token_lora_indices = self._get_token_lora_indices(x)
    bgmv_expand(x, lora_b_stacked, y, token_lora_indices, add_inputs)

def add_lora_linear(self, y, x, lora_a_stacked, lora_b_stacked, scale, output_slices, *, buffer=None, **kwargs):
    if buffer is None:
        r = lora_b_stacked[0].size(-1)
        buffer = torch.zeros(
            (len(output_slices), x.size(0), r),
            dtype=x.dtype,
            device=x.device,
        )
    self.add_shrink(buffer, x, lora_a_stacked, scale, **kwargs)
    self.add_expand(y, buffer, lora_b_stacked, output_slices, add_inputs=True, **kwargs)

@property
def sampler_indices_padded(self) -> torch.Tensor:
    return self._sampler_indices_padded[:]
```
**EN:** Embedding is a direct expand call. Linear again composes shrink and expand, but keeps the temporary buffer in the input dtype instead of forcing `float32`. The `sampler_indices_padded` override exposes the whole preallocated tensor, which helps XPU graph capture cope with dynamic lengths.
**CN:** Embedding 直接走 expand。linear 同样通过 shrink+expand 组合实现，但其中间缓冲区保持输入 dtype，而不是强制转成 `float32`。`sampler_indices_padded` 的重载会暴露完整预分配张量，这有助于 XPU 图捕获处理动态长度。

### MoE routing alignment / MoE 路由对齐
```python
def moe_lora_align_block_size(
    self,
    topk_ids: torch.Tensor,
    num_tokens: int,
    block_size: int,
    num_experts: int,
    max_loras: int,
    adapter_enabled: torch.Tensor,
    expert_map: torch.Tensor | None = None,
    pad_sorted_ids: bool = False,
    naive_block_assignment: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    (token_lora_mapping, _, _, _, lora_ids, _, _) = self.token_mapping_meta.meta_args(
        num_tokens, self.lora_config.specialize_active_lora
    )
    if naive_block_assignment:
        expert_ids = topk_ids.reshape(-1)
        sorted_ids = None
        num_tokens_post_pad = None
```
**EN:** The XPU backend computes LoRA-aware expert bucketing similarly to GPU. When the full path is taken, a custom op fills sorted token ids and padded counts, and an optional `expert_map` remaps expert ids after bucketing.
**CN:** XPU 后端与 GPU 类似，也会构造带 LoRA 感知的 expert 分桶。走完整路径时，自定义算子会填充排序后的 token id 和补齐后的计数；如果提供 `expert_map`，还会在分桶后重映射 expert id。

### Fused MoE execution / 融合式 MoE 执行
```python
(
    token_lora_mapping_meta,
    _,
    _,
    _,
    lora_ids,
    _,
    num_active_loras,
) = self.token_mapping_meta.meta_args(
    x.size(0), self.lora_config.specialize_active_lora
)
if token_lora_mapping is None:
    token_lora_mapping = token_lora_mapping_meta
fused_moe_lora(
    y,
    x,
    lora_a_stacked,
    lora_b_stacked,
    topk_weights,
    sorted_token_ids,
    expert_ids,
    num_tokens_post_padded,
    token_lora_mapping,
    max_lora_rank,
    top_k_num,
    lora_ids,
    num_active_loras,
    adapter_enabled,
    shrink_config.get("BLOCK_SIZE_M", 64),
    shrink_config.get("BLOCK_SIZE_N", 64),
    shrink_config.get("BLOCK_SIZE_K", 32),
    shrink_config.get("GROUP_SIZE_M", 8),
    shrink_config.get("NUM_WARPS", 4),
    shrink_config.get("NUM_STAGES", 3),
    shrink_config.get("SPLIT_K", 1),
    expand_config.get("BLOCK_SIZE_M", 64),
    expand_config.get("BLOCK_SIZE_N", 64),
    expand_config.get("BLOCK_SIZE_K", 32),
    expand_config.get("GROUP_SIZE_M", 8),
    expand_config.get("NUM_WARPS", 4),
    expand_config.get("NUM_STAGES", 3),
    expand_config.get("SPLIT_K", 1),
    mul_routed_weight,
    fully_sharded,
    offset,
)
```
**EN:** Despite using XPU kernels for standard LoRA paths, the fused MoE path still funnels through the shared `fused_moe_lora` launcher and the same metadata layout used by the Triton implementation.
**CN:** 虽然标准 LoRA 路径使用的是 XPU kernel，但融合式 MoE 路径仍然通过共享的 `fused_moe_lora` 启动器，并沿用 Triton 实现使用的元数据布局。

### W13 and W2 orchestration / W13 与 W2 调度
```python
if use_tuned_config:
    shrink_config = get_lora_op_configs(
        op_type="fused_moe_lora_w13_shrink",
        max_loras=max_loras,
        batch=num_tokens,
        hidden_size=x.shape[-1],
        rank=max_lora_rank,
        num_slices=num_slices,
        moe_intermediate_size=lora_b_stacked[0].shape[-2],
    )
    expand_config = get_lora_op_configs(
        op_type="fused_moe_lora_w13_expand",
        max_loras=max_loras,
        batch=num_tokens,
        hidden_size=x.shape[-1],
        rank=max_lora_rank,
        num_slices=num_slices,
        moe_intermediate_size=lora_b_stacked[0].shape[-2],
    )
else:
    shrink_config = get_config(op_type="fused_moe_lora_w13_shrink")
    expand_config = get_config(op_type="fused_moe_lora_w13_expand")
```
**EN:** The orchestration layer mirrors the GPU design: choose tuned or heuristic configs, prepare LoRA-aware routing for W13, then reuse the resulting tensors when W2 runs. In the real `add_lora_w2`, `offset = shard_size * tp_rank` shifts the output window for fully sharded tensor-parallel weights.
**CN:** 这层调度逻辑与 GPU 设计基本一致：先为 W13 选择调优配置或启发式配置，并准备带 LoRA 感知的路由；随后 W2 会复用这些张量。真实的 `add_lora_w2` 中，`offset = shard_size * tp_rank` 用于在完全分片的张量并行权重上平移输出窗口。

## Key Concepts / 关键概念
- Dynamic-shape friendliness for XPU compilation. / 面向 XPU 编译的动态形状支持。
- Narrowed views instead of repeated metadata copies. / 通过裁剪视图替代重复的元数据拷贝。
- Shared fused MoE path with backend-specific front-end ops. / 前端基础算子按后端区分，但融合式 MoE 路径共享。

## Dependencies / 依赖关系
- `vllm.lora.ops.xpu_ops`: `bgmv_shrink`, `bgmv_expand`, `bgmv_expand_slice`
- `vllm.lora.ops.triton_ops`: `LoRAKernelMeta`, `fused_moe_lora`
- `vllm._custom_ops` for MoE block alignment
- `PunicaWrapperBase`
