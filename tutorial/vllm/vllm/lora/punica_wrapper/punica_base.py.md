# punica_base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/punica_wrapper/punica_base.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Define the common Punica wrapper interface and shared metadata management for multi-LoRA execution. / [CN] 定义 Punica wrapper 的通用接口，以及多 LoRA 执行所需的共享元数据管理逻辑。

## Line-by-Line Analysis / 逐行分析
### Abstract interface / 抽象接口
```python
class PunicaWrapperABC(ABC):
    @abstractmethod
    def update_metadata(
        self,
        mapping: "LoRAMapping",
        lora_index_to_id: list[int | None],
        max_loras: int,
        vocab_size: int,
        **kwargs,
    ) -> None:
        raise NotImplementedError

    @abstractmethod
    def add_shrink(
        self,
        y: tuple[torch.Tensor, ...] | torch.Tensor,
        x: torch.Tensor,
        lora_a_stacked: tuple[torch.Tensor, ...],
        scale: float,
        **kwargs,
    ) -> torch.Tensor | None:
        raise NotImplementedError
```
**EN:** `PunicaWrapperABC` formalizes the operations every backend must support: metadata refresh plus the main LoRA kernels (`shrink`, `expand`, embedding, linear, logits, and MoE variants). The detailed docstrings in this file define the mathematical semantics independent of hardware.
**CN:** `PunicaWrapperABC` 规定了每个后端都必须实现的能力：元数据更新，以及 LoRA 的核心算子（`shrink`、`expand`、embedding、linear、logits 和 MoE 变体）。本文件中的详细文档字符串给出了与硬件无关的数学语义。

### Preallocated state buffers / 预分配状态缓冲区
```python
def __init__(
    self,
    max_num_batched_tokens: int,
    max_batches: int,
    device: torch.device | str,
    **kwargs,
):
    self._token_lora_indices = torch.empty(
        max_num_batched_tokens, dtype=torch.long, device=device
    )
    self._sampler_indices = torch.empty(
        max_num_batched_tokens, dtype=torch.long, device=device
    )
    self._sampler_indices_padded = torch.empty(
        max_num_batched_tokens, dtype=torch.long, device=device
    )
    self._embeddings_indices = torch.empty(
        2, max_num_batched_tokens, dtype=torch.long, device=device
    )
    self.indices_len: list[int | None] = [None] * 4
    self._seq_start_locs = torch.empty(max_batches, dtype=torch.long, device=device)
    self._seq_lengths = torch.empty(max_batches, dtype=torch.long, device=device)
    self._lora_indices_per_batch = torch.empty(
        max_batches, dtype=torch.long, device=device
    )
    self.device: torch.device = device
    self.max_length: int = 0
    self.token_nums: int = 0
    self.batch_size: int = -1
    self.is_prefill = False
    self.no_lora = False
```
**EN:** The base class preallocates all recurring metadata buffers once on the target device. Backends then slice these buffers by active length instead of reallocating per batch, reducing control-plane overhead.
**CN:** 基类会在目标设备上一次性预分配所有常用元数据缓冲区。后端随后只需按当前有效长度切片，而不必为每个 batch 重新分配，从而降低控制路径开销。

### Base mapping refresh / 基础映射更新
```python
def _update_base_metadata(
    self,
    mapping: "LoRAMapping",
    lora_index_to_id: list[int | None],
    max_loras: int,
    vocab_size: int,
):
    extra_vocab_size = 0
    (
        base_indices,
        sampler_indices,
        sampler_indices_padded,
        embeddings_indices,
        indices_len,
    ) = convert_mapping(
        mapping,
        lora_index_to_id,
        max_loras,
        vocab_size,
        extra_vocab_size,
        self.device,
    )
    self._token_lora_indices[: base_indices.shape[0]].copy_(base_indices)
    self._sampler_indices[: sampler_indices.shape[0]].copy_(sampler_indices)
    self._sampler_indices_padded[: sampler_indices_padded.shape[0]].copy_(
        sampler_indices_padded
    )
    self._embeddings_indices[
        : embeddings_indices.shape[0], : embeddings_indices.shape[1]
    ].copy_(embeddings_indices)
    self.indices_len[:] = indices_len
```
**EN:** `_update_base_metadata` calls `convert_mapping` and copies the resulting tensors into the preallocated buffers. The code explicitly fixes `extra_vocab_size = 0`, reflecting the current removal of LoRA extra-vocabulary support.
**CN:** `_update_base_metadata` 调用 `convert_mapping`，再把结果复制到预分配缓冲区中。代码显式把 `extra_vocab_size` 固定为 `0`，说明当前版本已经暂时移除了 LoRA 扩展词表支持。

### Prefill grouping metadata / Prefill 分组元数据
```python
def _update_prefill_metadata(self, token_lora_tensor: torch.Tensor) -> None:
    (
        b_seq_start_tensor,
        seq_length_tensor,
        lora_indices_tensor,
        batch_size,
        max_length,
        token_nums,
        no_lora,
    ) = compute_meta(token_lora_tensor)

    self._seq_start_locs[: b_seq_start_tensor.shape[0]].copy_(b_seq_start_tensor)
    self._seq_lengths[: seq_length_tensor.shape[0]].copy_(seq_length_tensor)
    self._lora_indices_per_batch[: lora_indices_tensor.shape[0]].copy_(
        lora_indices_tensor
    )
    self.batch_size = batch_size
    self.max_length = max_length
    self.token_nums = token_nums
    self.no_lora = no_lora

@property
def prefill_metadata(self) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, int, int, int]:
    return (
        self._seq_start_locs[: self.batch_size],
        self._seq_lengths[: self.batch_size],
        self._lora_indices_per_batch[: self.batch_size],
        self.batch_size,
        self.max_length,
        self.token_nums,
    )
```
**EN:** During prefill, consecutive tokens with the same LoRA id are compacted into segment metadata. The `prefill_metadata` property exposes exactly the tuple expected by segment-based kernels such as SGMV.
**CN:** 在 prefill 阶段，连续使用同一 LoRA id 的 token 会被压缩成分段元数据。`prefill_metadata` 属性返回的正是 SGMV 这类按段执行的 kernel 所需参数元组。

### Public metadata access / 对外元数据访问
```python
@property
def token_lora_indices(self) -> torch.Tensor:
    token_lora_len = self.indices_len[0]
    return self._token_lora_indices[:token_lora_len]

@property
def sampler_indices(self) -> torch.Tensor:
    sampler_indices_len = self.indices_len[1]
    return self._sampler_indices[:sampler_indices_len]

def update_metadata(
    self,
    mapping: "LoRAMapping",
    lora_index_to_id: list[int | None],
    max_loras: int,
    vocab_size: int,
    **kwargs,
):
    self._update_base_metadata(mapping, lora_index_to_id, max_loras, vocab_size)
    if mapping.is_prefill:
        self._update_prefill_metadata(self.token_lora_indices)
        self.is_prefill = True
    else:
        self.is_prefill = False
```
**EN:** Accessor properties slice cached tensors down to their live region. `update_metadata` is the shared control entrypoint: it always refreshes token/request mappings, and conditionally prepares extra prefill metadata.
**CN:** 这些访问器会把缓存张量裁剪到当前有效区域。`update_metadata` 是共享的控制入口：它总会刷新 token/请求映射，并在 prefill 模式下额外准备分段元数据。

### Backend extension points / 后端扩展点
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
    raise NotImplementedError

def add_lora_fused_moe(
    self,
    y: torch.Tensor,
    x: torch.Tensor,
    lora_a_stacked: tuple[torch.Tensor, ...],
    lora_b_stacked: tuple[torch.Tensor, ...],
    topk_weights: torch.Tensor,
    sorted_token_ids: torch.Tensor | None,
    expert_ids: torch.Tensor,
    num_tokens_post_padded: torch.Tensor | None,
    max_lora_rank: int,
    top_k_num: int,
    shrink_config,
    expand_config,
    adapter_enabled: torch.Tensor,
    mul_routed_weight=False,
    fully_sharded: bool = False,
    offset: int = 0,
    token_lora_mapping: torch.Tensor | None = None,
):
    raise NotImplementedError
```
**EN:** The lower part of the base class reserves hooks for MoE-specific routing and fused execution; the same pattern continues for `add_lora_w13` and `add_lora_w2`. Concrete backends fill these in because routing layout and kernel launch details are device-specific.
**CN:** 基类的后半部分为 MoE 专用的路由与融合执行预留了接口；`add_lora_w13` 和 `add_lora_w2` 也沿用同样模式。之所以交给具体后端实现，是因为路由布局和 kernel 启动细节都与设备强相关。

## Key Concepts / 关键概念
- Hardware-agnostic LoRA wrapper contract. / 与硬件无关的 LoRA wrapper 契约。
- Reusable device-resident metadata buffers. / 可复用的设备端元数据缓冲区。
- Separate metadata paths for decode vs. prefill. / 为 decode 与 prefill 维护不同的元数据路径。
- MoE routing is delegated to backend-specific implementations. / MoE 路由交给各后端定制实现。

## Dependencies / 依赖关系
- `torch`
- `abc.ABC` / `abstractmethod`
- `compute_meta` and `convert_mapping` from `utils.py`
- `LoRAMapping` (type checking)
