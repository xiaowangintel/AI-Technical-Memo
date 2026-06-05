# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/punica_wrapper/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Build tensor metadata that maps tokens and requests to active LoRA adapters. / [CN] 构造张量元数据，把 token 和请求映射到当前激活的 LoRA adapter。

## Line-by-Line Analysis / 逐行分析
### Prefill metadata compaction / Prefill 元数据压缩
```python
def compute_meta(
    token_lora_tensor: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, int, int, int, bool]:
    lora_indices_tensor, seq_length_tensor = torch.unique_consecutive(
        token_lora_tensor, return_counts=True
    )
    cum_result = torch.cumsum(seq_length_tensor, dim=0)
    b_seq_start_tensor = torch.zeros_like(seq_length_tensor)
    b_seq_start_tensor[1:].copy_(cum_result[:-1])
    max_length = seq_length_tensor.max().item()
    token_nums = seq_length_tensor.sum().item()
    batch_size = lora_indices_tensor.size(0)
    no_lora = False
    if batch_size == 1 and lora_indices_tensor == -1:
        no_lora = True
    return (
        b_seq_start_tensor,
        seq_length_tensor,
        lora_indices_tensor,
        batch_size,
        max_length,
        token_nums,
        no_lora,
    )
```
**EN:** `compute_meta` merges consecutive tokens that use the same LoRA id, producing compact sequence starts, lengths, and per-segment adapter ids. This lets prefill kernels run on grouped segments instead of per-token metadata. The `no_lora` fast path skips kernel launches when the whole batch uses adapter `-1`.
**CN:** `compute_meta` 会把连续使用相同 LoRA id 的 token 合并，生成紧凑的起始位置、长度和分段 adapter id。这样 prefill kernel 可以按分段而不是逐 token 使用元数据。`no_lora` 快速路径用于在整批都是 `-1`（不启用 LoRA）时直接跳过 kernel。

### Mapping conversion / 映射转换
```python
def convert_mapping(
    mapping: "LoRAMapping",
    lora_index_to_id: list[int | None],
    max_loras: int,
    vocab_size: int,
    extra_vocab_size: int,
    device: torch.device,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor, list[int]]:
    index_mapping_indices: list[int] = list(mapping.index_mapping).copy()
    embedding_indices = index_mapping_indices.copy()
    lora_indices = index_mapping_indices.copy()

    prompt_mapping: list[int] = [
        lora_index_to_id.index(x) if x > 0 else -1 for x in mapping.prompt_mapping
    ]
    for i in range(len(index_mapping_indices)):
        lora_idx = (
            lora_index_to_id.index(index_mapping_indices[i])
            if index_mapping_indices[i] > 0
            else -1
        )
        embedding_indices[i] = lora_idx if index_mapping_indices[i] > 0 else 0
        lora_indices[i] = lora_idx
```
**EN:** This function converts logical adapter ids from `LoRAMapping` into dense runtime indices. It prepares one mapping for token-level kernels, one for prompt/logits processing, and one for embedding lookups.
**CN:** 该函数把 `LoRAMapping` 中的逻辑 adapter id 转成运行时稠密索引，分别为 token 级 kernel、prompt/logits 处理以及 embedding 查找准备不同映射。

### Tensor packing / 张量打包
```python
indices = torch.tensor(indices_list, dtype=torch.long, device=device)
prompt_mapping_tensor = torch.tensor(
    prompt_mapping, dtype=torch.long, device=device
)
embeddings_indices = torch.stack(
    [
        indices[2] * extra_vocab_size,
        indices[2] * (vocab_size + extra_vocab_size),
    ]
)
embeddings_indices = torch.where(
    embeddings_indices == -1, max_loras - 1, embeddings_indices
)
base_indices = indices[1]
sampler_indices = prompt_mapping_tensor
sampler_indices_padded = sampler_indices.clone()
sampler_indices_padded = torch.where(
    sampler_indices_padded == -1, max_loras - 1, sampler_indices_padded
)
sampler_indices_padded = torch.arange(
    0, len(sampler_indices_padded), device=device, dtype=torch.long
) + (sampler_indices_padded * len(sampler_indices_padded))
```
**EN:** Packed tensors are moved to the target device once and reused by wrappers. Negative adapter ids are rewritten to a sentinel slot (`max_loras - 1`) for padded indexing, and `sampler_indices_padded` flattens `(adapter, position)` into one index for efficient sampler lookup.
**CN:** 打包后的张量一次性搬到目标设备，后续由 wrapper 复用。负 adapter id 会被改写到哨兵槽位（`max_loras - 1`）以支持填充索引，而 `sampler_indices_padded` 把 `(adapter, position)` 压成单一索引，便于 sampler 高效访问。

## Key Concepts / 关键概念
- Consecutive-run compression for prefill kernels. / 为 prefill kernel 做连续段压缩。
- Dense runtime indexing of LoRA adapters. / 将 LoRA adapter 转成稠密运行时索引。
- Sentinel padding for “no LoRA” cases. / 用哨兵索引表示“无 LoRA”情况。

## Dependencies / 依赖关系
- `torch`
- `LoRAMapping` (type checking only)
- Consumers in `punica_base.py` and backend wrappers
