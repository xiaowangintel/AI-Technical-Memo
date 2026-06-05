# punica_gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/punica_wrapper/punica_gpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implement the CUDA/Triton Punica backend, including specialized metadata handling and fused MoE LoRA kernels. / [CN] 实现 CUDA/Triton 版 Punica 后端，包括专用元数据处理与融合式 MoE LoRA kernel。

## Line-by-Line Analysis / 逐行分析
### Initialization and kernel metadata / 初始化与 kernel 元数据
```python
def __init__(
    self,
    max_num_batched_tokens: int,
    max_batches: int,
    device: torch.device | str,
    **kwargs,
):
    PunicaWrapperBase.__init__(self, max_num_batched_tokens, max_batches, device)

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
**EN:** The GPU wrapper extends the shared buffers with Triton-specific metadata objects. `captured_lora_counts` enables CUDA-graph specialization for different active-LoRA counts, and separate metadata is maintained for token mapping and prompt/logits mapping.
**CN:** GPU wrapper 在共享缓冲区之外，还创建了 Triton 专用的元数据对象。`captured_lora_counts` 用于针对不同激活 LoRA 数量做 CUDA Graph 特化，同时分别维护 token 映射和 prompt/logits 映射的元数据。

### Metadata refresh and core kernels / 元数据刷新与核心 kernel
```python
def update_metadata(self, mapping: LoRAMapping, lora_index_to_id, max_loras, vocab_size, **kwargs):
    self.is_prefill = mapping.is_prefill
    self._update_base_metadata(mapping, lora_index_to_id, max_loras, vocab_size)
    self.token_mapping_meta.prepare_tensors(self.token_lora_indices)
    self.prompt_mapping_meta.prepare_tensors(self.sampler_indices)

def add_shrink(self, y, x, lora_a_stacked, scale: float, **kwargs):
    x = x.view(-1, x.shape[-1])
    lora_shrink(
        x,
        lora_a_stacked,
        y,
        *self.token_mapping_meta.meta_args(
            x.size(0), self.lora_config.specialize_active_lora
        ),
        scale,
    )

def add_expand(self, y, x, lora_b_stacked, output_slices, offset_start: int = 0, add_inputs=True, **kwargs):
    y_org = y
    y = y.view(-1, y.shape[-1])
    num_tokens = x.size(1)
    lora_expand(
        x,
        lora_b_stacked,
        y,
        *self.token_mapping_meta.meta_args(
            num_tokens, self.lora_config.specialize_active_lora
        ),
        offset_start=offset_start,
        add_inputs=add_inputs,
    )
    y = y.view_as(y_org)
```
**EN:** `update_metadata` prepares the compact mapping tensors consumed by Triton kernels. The standard shrink/expand paths delegate directly to `lora_shrink` and `lora_expand`, with metadata arguments derived from the live batch size.
**CN:** `update_metadata` 会准备 Triton kernel 所需的紧凑映射张量。标准的 shrink/expand 路径直接委托给 `lora_shrink` 与 `lora_expand`，其元数据参数根据当前 batch 的实际大小生成。

### Embedding, linear, and logits helpers / Embedding、linear 与 logits 辅助接口
```python
def add_lora_embedding(self, y, x, lora_b_stacked, add_inputs: bool = True, **kwargs):
    lora_expand(
        x.unsqueeze(dim=0),
        (lora_b_stacked,),
        y,
        *self.token_mapping_meta.meta_args(
            x.size(0), self.lora_config.specialize_active_lora
        ),
        offset_start=0,
        add_inputs=add_inputs,
    )

def add_lora_linear(self, y, x, lora_a_stacked, lora_b_stacked, scale, output_slices, *, buffer=None, **kwargs):
    r = lora_b_stacked[0].size(-1)
    buffer = torch.empty(
        (len(output_slices), x.size(0), r), dtype=torch.float32, device=x.device
    )
    add_inputs = kwargs.pop("add_inputs", True)
    self.add_shrink(buffer, x, lora_a_stacked, scale, **kwargs)
    self.add_expand(y, buffer, lora_b_stacked, output_slices, add_inputs=add_inputs, **kwargs)

def add_lora_logits(self, y, x, lora_a_stacked, lora_b_stacked, scale, *, buffer=None, **kwargs):
    x = x.view(-1, x.shape[-1])
    buffer = torch.empty((x.size(0), lora_b_stacked.size(-1)), dtype=torch.float32, device=x.device)
    lora_shrink(
        x,
        [lora_a_stacked],
        buffer.unsqueeze(dim=0),
        *self.prompt_mapping_meta.meta_args(
            x.size(0), self.lora_config.specialize_active_lora
        ),
        scale,
    )
```
**EN:** Embedding is treated as a single-slice expand. Linear and logits paths allocate temporary rank-space buffers in `float32`, then compose shrink and expand using token-level metadata for activations and prompt-level metadata for sampler/logits behavior.
**CN:** Embedding 被视作单切片 expand。linear 与 logits 路径会先分配 `float32` 的低秩临时缓冲区，再组合 shrink 与 expand；其中激活使用 token 级元数据，而 sampler/logits 使用 prompt 级元数据。

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
    token_lora_mapping: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    (token_lora_mapping_meta, _, _, _, lora_ids, _, _) = self.token_mapping_meta.meta_args(
        num_tokens, self.lora_config.specialize_active_lora
    )
    if token_lora_mapping is None:
        token_lora_mapping = token_lora_mapping_meta
    kernel_num_experts = (
        expert_map.numel() if expert_map is not None else num_experts
    )
    if naive_block_assignment:
        expert_ids = topk_ids.reshape(-1)
        sorted_ids = None
        num_tokens_post_pad = None
```
**EN:** This method prepares routing buffers for fused MoE execution. It optionally accepts an externally supplied token→LoRA map (used in expert parallelism), sizes expert buckets carefully for global expert ids, and can fall back to a trivial assignment when sparsity is high.
**CN:** 该方法为融合式 MoE 执行准备路由缓冲区。它可以接收外部传入的 token→LoRA 映射（用于 expert parallelism），并在存在全局 expert id 时正确设置桶大小；当稀疏度较高时，也可以退化为简单分配路径。

### Fused MoE launch / 融合式 MoE 启动
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
**EN:** `add_lora_fused_moe` is a thin launcher around the Triton fused kernel. It injects the active LoRA set, routing tensors, and many tuning parameters for both the shrink and expand phases.
**CN:** `add_lora_fused_moe` 是对 Triton 融合 kernel 的轻量封装，它会把激活 LoRA 集合、路由张量以及 shrink/expand 两阶段的大量调优参数统一传入。

### W13 path with routing reuse / W13 路径与路由复用
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
    get_config = functools.partial(
        try_get_optimal_moe_lora_config,
        w1_shape=w1.shape,
        w2_shape=w2.shape,
        rank=max_lora_rank,
        top_k=top_k,
        dtype=config_dtype,
        M=num_tokens,
    )
    shrink_config = get_config(op_type="fused_moe_lora_w13_shrink")
    expand_config = get_config(op_type="fused_moe_lora_w13_expand")
```
**EN:** `add_lora_w13` first chooses tuned or heuristic kernel configs for the W1/W3 branch. The real function then aligns LoRA-aware expert routing, launches `add_lora_fused_moe`, and returns routing tensors for the following W2 step.
**CN:** `add_lora_w13` 首先为 W1/W3 分支选择调优配置或启发式配置。真实函数随后会对齐带 LoRA 感知的 expert 路由、调用 `add_lora_fused_moe`，并把路由张量返回给后续的 W2 步骤复用。

### W2 path / W2 路径
```python
_sorted = sorted_token_ids_lora
_eids = expert_ids_lora
if _sorted is not None:
    assert _eids is not None
    _eids = _eids.view(max_loras, -1)
    _sorted = _sorted.view(max_loras, -1)

shard_size = lora_b_stacked[0].shape[-2]
offset = shard_size * tp_rank if fully_sharded else 0

self.add_lora_fused_moe(
    y,
    x,
    lora_a_stacked,
    lora_b_stacked,
    topk_weights,
    _sorted,
    _eids,
    num_tokens_post_padded_lora,
    max_lora_rank,
    top_k,
    shrink_config,
    expand_config,
    adapter_enabled,
    True,
    fully_sharded=fully_sharded,
    offset=offset,
    token_lora_mapping=token_lora_mapping,
)
```
**EN:** The W2 branch reuses routing metadata from `add_lora_w13`, reshapes it for the fused kernel, and applies an output offset when LoRA-B is tensor-parallel sharded. Passing `True` for `mul_routed_weight` folds routed expert weights into the fused computation.
**CN:** W2 分支会复用 `add_lora_w13` 返回的路由元数据，并在进入融合 kernel 前调整其形状；如果 LoRA-B 是张量并行切分的，还会施加输出偏移。把 `mul_routed_weight` 设为 `True` 表示在融合计算中直接乘入路由权重。

## Key Concepts / 关键概念
- Triton metadata specialization for active LoRA counts. / 针对激活 LoRA 数量的 Triton 元数据特化。
- Token mapping vs. prompt mapping. / token 映射与 prompt 映射分离。
- Fused MoE LoRA execution with reusable routing tensors. / 通过可复用途由张量执行融合式 MoE LoRA。
- Tuned configs can override heuristic kernel selection. / 调优配置可以覆盖启发式 kernel 选择。

## Dependencies / 依赖关系
- `vllm.lora.ops.triton_ops`: `LoRAKernelMeta`, `lora_shrink`, `lora_expand`, `fused_moe_lora`
- `vllm._custom_ops` for MoE block alignment
- `get_captured_lora_counts`, `round_up`, Triton utilities
- `PunicaWrapperBase`
