# fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/layers/fused_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Adds LoRA support to fused MoE layers, including expert-parallel constraints, stacked expert-wise weights, and a specialized 3D variant. / 为 fused MoE 层提供 LoRA 支持，包括 expert parallel 约束、按专家堆叠的权重，以及专门的 3D 变体。

## Line-by-Line Analysis / 逐行分析
### Constructor and kernel integration
```python
class FusedMoEWithLoRA(BaseLayerWithLoRA):
    def __init__(self, base_layer: FusedMoE) -> None:
        self.base_layer = base_layer
        self._ep_check()
        self.tp_size = self.base_layer.tp_size
        self.tp_rank = self.base_layer.tp_rank
        self.device = _get_lora_device(base_layer)
        self._w13_slices = 2 if base_layer.moe_config.is_act_and_mul else 1
        self.n_slices = base_layer.local_num_experts * (self._w13_slices + 1)
        ...
        moe_kernel = ...
        assert moe_kernel.supports_lora()
        self._moe_kernel = moe_kernel
        self.base_layer._replace_quant_method(
            FusedMoEModularMethod(self.base_layer.quant_method, moe_kernel)
        )
```
**EN:** The constructor binds LoRA to a fused MoE layer, derives TP metadata from the MoE-aware base layer, and chooses how many W1/W3 slices exist depending on whether the MoE is gated. It then ensures the selected fused-expert kernel supports LoRA and wraps the original quantization method with a modular variant that can consume LoRA context.
**CN:** 构造函数把 LoRA 绑定到 fused MoE 层上，从带有 MoE 语义的底层层对象中取得 TP 元信息，并根据 MoE 是否带门控决定 W1/W3 需要多少个 slice。随后它确认所选 fused-expert kernel 支持 LoRA，并用可接收 LoRA 上下文的模块化量化方法替换原量化实现。

### Building MoE-specific LoRA context
```python
def _build_lora_context(self):
    return MoELoRAContext(
        w13_lora_a_stacked=self.w13_lora_a_stacked,
        w13_lora_b_stacked=self.w13_lora_b_stacked,
        w2_lora_a_stacked=self.w2_lora_a_stacked,
        w2_lora_b_stacked=self.w2_lora_b_stacked,
        adapter_enabled=self.adapter_enabled,
        max_loras=self.max_loras,
        top_k=self.base_layer.top_k,
        ...
    )
```
**EN:** Instead of applying LoRA directly in Python, the layer packages all runtime tensors and flags into a `MoELoRAContext`. The fused experts kernel later reads this context to apply expert-specific adapters during routing and expert execution.
**CN:** 这里并不是直接在 Python 中执行 LoRA，而是把所有运行时张量与开关打包成 `MoELoRAContext`。之后 fused experts 内核会读取该上下文，在路由和专家计算阶段应用每个专家对应的适配器。

### Allocating expert-wise stacked weights
```python
def _create_lora_a_weights(self, max_loras, lora_config):
    self.w13_lora_a_stacked = tuple(torch.zeros((max_loras, local_num_experts, ..., hidden_size), ...)
                                    for _ in range(self._w13_slices))
    self.w2_lora_a_stacked = (torch.zeros((max_loras, local_num_experts, max_rank, intermediate_size_per_partition), ...),)

def _create_lora_b_weights(self, max_loras, lora_config):
    self.w13_lora_b_stacked = tuple(torch.zeros((max_loras, local_num_experts, intermediate_size_per_partition, max_rank), ...)
                                    for _ in range(self._w13_slices))
    self.w2_lora_b_stacked = (torch.zeros((max_loras, local_num_experts, hidden_size_or_shard, max_rank), ...),)
```
**EN:** LoRA weights are stored per adapter and per local expert. W1/W3 use one or two stacks depending on MoE type, while W2 always uses one stack. Fully sharded mode changes the rank or output dimensions so each TP worker stores only its share.
**CN:** LoRA 权重按“适配器 ID × 本地专家”组织存储。W1/W3 根据 MoE 类型使用一个或两个堆栈，而 W2 始终只有一个堆栈。fully sharded 模式会调整 rank 或输出维度，使每个 TP worker 只保存自己的那一份。

### EP and fully-sharded safety checks
```python
def _ep_check(self):
    if self.base_layer.use_ep:
        assert all2all_backend == "allgather_reducescatter"
        assert not moe_config.moe_parallel_config.is_sequence_parallel

def _verify_ep_fs(self, lora_config: LoRAConfig):
    assert not (self.base_layer.use_ep and lora_config.fully_sharded_loras)
```
**EN:** The file explicitly guards unsupported distributed combinations. Expert parallelism is only supported with a specific communication backend, and it cannot be combined with fully sharded LoRA because both features partition state across the same TP group in incompatible ways.
**CN:** 本文件显式拦截不受支持的分布式组合。Expert parallel 仅支持特定通信后端，同时也不能与 fully sharded LoRA 结合使用，因为两者都会在同一个 TP 组上切分状态，但假设不兼容。

### Initializing runtime-visible LoRA storage
```python
def create_lora_weights(self, max_loras, lora_config, model_config=None):
    self._verify_ep_fs(lora_config)
    self.max_loras = lora_config.max_loras
    self.fully_sharded = lora_config.fully_sharded_loras
    self.adapter_enabled = torch.tensor([0] * (max_loras + 1), dtype=torch.int, device=self.device)
    self._create_lora_a_weights(max_loras, lora_config)
    self._create_lora_b_weights(max_loras, lora_config)
    self.lora_a_stacked = []
    self.lora_b_stacked = []
    for lora_id in range(max_loras):
        for experts_id in range(self.base_layer.local_num_experts):
            ... append w1 / w2 / optional w3 views ...
```
**EN:** `create_lora_weights` allocates the real expert-wise tensors and also builds flattened `lora_a_stacked` / `lora_b_stacked` lists for compatibility with generic LoRA utilities such as dummy-weight creation. The flattened order mirrors the expert/slice layout used at runtime.
**CN:** `create_lora_weights` 会先分配真正按专家组织的 LoRA 张量，同时还构造扁平化的 `lora_a_stacked` / `lora_b_stacked` 列表，以兼容通用 LoRA 工具（例如 dummy 权重创建）。这个扁平顺序与运行时的专家/slice 布局一致。

### Tensor-parallel slicing helpers
```python
def _slice_w13_a(self, w13_lora_a):
    if self.tp_size == 1 or not self.fully_sharded:
        return w13_lora_a
    return w13_lora_a[:, start_idx:end_idx, :]

def _slice_w13_b(self, w13_lora_b):
    if self.tp_size == 1:
        return w13_lora_b
    return w13_lora_b[:, start_idx:end_idx, :]

def _slice_w2_a(self, w2_lora_a):
    return w2_lora_a[:, :, start_idx:end_idx]

def _slice_w2_b(self, w2_lora_b):
    if self.tp_size == 1 or not self.fully_sharded:
        return w2_lora_b
    return w2_lora_b[:, start_idx:end_idx, :]
```
**EN:** These helpers encode how each MoE projection is partitioned across tensor-parallel ranks. W1/W3 and W2 do not shard in the same dimension, and fully sharded mode only affects the dimensions that correspond to LoRA rank or final hidden output.
**CN:** 这些辅助函数定义了各个 MoE 投影在张量并行中的切分方式。W1/W3 与 W2 的切分维度并不相同，而 fully sharded 模式只会影响对应 LoRA rank 或最终 hidden 输出的那些维度。

### Resetting and loading adapter weights
```python
def reset_lora(self, index: int):
    for pos in range(self._w13_slices):
        self.w13_lora_a_stacked[pos][index] = 0
        self.w13_lora_b_stacked[pos][index] = 0
    self.w2_lora_a_stacked[0][index] = 0
    self.w2_lora_b_stacked[0][index] = 0
    self.adapter_enabled[index] = 0

def set_lora(self, index, lora_a, lora_b):
    self.reset_lora(index)
    self.adapter_enabled[index] = 1
    ... slice w1 / w2 / optional w3 ...
    ... copy_ into stacked tensors ...
```
**EN:** Adapter updates are destructive overwrites: existing tensors are zeroed, the adapter is marked enabled, and incoming LoRA weights are TP-sliced before asynchronous copies into the stacked buffers. The code assumes EP slicing already happened upstream, so local expert counts must already match.
**CN:** 适配器更新采用覆盖式写入：先把已有张量清零，再标记适配器启用，然后对输入 LoRA 权重进行 TP 切分并异步拷贝到堆叠缓冲区。代码假设 EP 切分已经在上游完成，因此本地专家数量必须已经匹配。

### Wiring the context into execution
```python
def set_mapping(self, punica_wrapper):
    super().set_mapping(punica_wrapper)
    lora_context = self._build_lora_context()
    self._moe_kernel.fused_experts.set_lora_context(lora_context)
    prepare_finalize = self._moe_kernel.prepare_finalize
    if hasattr(prepare_finalize, "set_lora_context"):
        prepare_finalize.set_lora_context(lora_context)

def forward(self, *args, **kwargs):
    return self.base_layer.forward(*args, **kwargs)
```
**EN:** Once the Punica mapping is known, the layer pushes the constructed context down into both the fused experts kernel and the optional prepare/finalize stage. The wrapper’s own `forward` simply delegates to the base MoE layer, which now runs with LoRA-aware kernels.
**CN:** 在 Punica 映射准备好之后，该层会把构造出的上下文下发给 fused experts 内核以及可选的 prepare/finalize 阶段。包装器自身的 `forward` 只是转发到底层 MoE 层，而此时底层已经切换为支持 LoRA 的内核路径。

### 3D variant for combined W1/W3 layouts
```python
class FusedMoE3DWithLoRA(FusedMoEWithLoRA):
    def __init__(self, base_layer):
        super().__init__(base_layer)
        self._w13_slices = 1

    def _slice_w13_b(self, w13_lora_b: torch.Tensor):
        if self._base_model == "GptOssForCausalLM":
            w1_lora_b = w13_lora_b[:, ::2, :]
            w3_lora_b = w13_lora_b[:, 1::2, :]
            ... interleave after slicing ...
        else:
            ... split first half / second half and concat ...
```
**EN:** `FusedMoE3DWithLoRA` handles models that store W1 and W3 together as a single LoRA tensor. Its custom `_slice_w13_b` supports both standard concatenated layouts and GPT-OSS’s interleaved layout, then `set_lora` writes the two logical projections through a single combined buffer.
**CN:** `FusedMoE3DWithLoRA` 用于处理把 W1 和 W3 合并存储为单个 LoRA 张量的模型。它自定义的 `_slice_w13_b` 同时支持普通拼接布局和 GPT-OSS 的交错布局，然后 `set_lora` 通过一个合并缓冲区写入这两个逻辑投影。

### Size helpers and replacement policy
```python
@property
def w13_output_size(self):
    return self.w13_lora_b_stacked[0].shape[-2] * self.tp_size

@classmethod
def can_replace_layer(cls, source_layer, lora_config, packed_modules_list, model_config=None):
    moe_cls = maybe_get_oot_by_class(FusedMoE)
    return isinstance(source_layer, moe_cls) and len(packed_modules_list) == 1
```
**EN:** The 3D variant exposes helper properties that report full logical input/output sizes even when tensors are stored in sharded form. Its replacement rule differentiates it from the base MoE wrapper by expecting a single packed module group instead of the W1/W2/W3-style split.
**CN:** 3D 变体提供了一组辅助属性，用于在内部张量是分片存储时依然返回完整的逻辑输入/输出尺寸。它的替换规则也与基础 MoE 包装器不同：这里期望的是单个打包模块组，而不是 W1/W2/W3 那种拆分形式。

## Key Concepts / 关键概念
- **Expert-wise LoRA storage**: weights are organized by adapter ID and local expert, not just by layer. / **按专家组织的 LoRA 存储**：权重不是只按层存，而是按适配器 ID 与本地专家共同组织。
- **Kernel-driven execution**: LoRA is applied through fused MoE kernels using `MoELoRAContext`. / **内核驱动执行**：LoRA 通过 fused MoE 内核和 `MoELoRAContext` 执行。
- **EP compatibility rules**: expert parallelism requires specific communication settings and forbids fully sharded LoRA. / **EP 兼容规则**：expert parallel 需要特定通信设置，并禁止与 fully sharded LoRA 同时启用。
- **Projection-specific sharding**: W1/W3 and W2 follow different TP slicing conventions. / **投影特定分片**：W1/W3 与 W2 使用不同的张量并行切分规则。
- **3D packed layout support**: some models pack W1/W3 together and need specialized slicing logic. / **3D 打包布局支持**：某些模型把 W1/W3 合并存储，需要专门的切分逻辑。

## Dependencies / 依赖关系
- `torch`, `torch.nn`: buffer allocation, zeroing, slicing, and asynchronous copies. / `torch`, `torch.nn`：缓冲区分配、清零、切分与异步拷贝。
- `LoRAConfig`, `PretrainedConfig`: adapter sizing and model-architecture metadata. / `LoRAConfig`, `PretrainedConfig`：适配器尺寸配置与模型架构元数据。
- `FusedMoE`, `FusedMoEModularMethod`, `FusedMoEKernel`: base MoE execution path and LoRA-capable fused kernels. / `FusedMoE`, `FusedMoEModularMethod`, `FusedMoEKernel`：基础 MoE 执行路径与支持 LoRA 的融合内核。
- `MoELoRAContext`, `MoEPrepareAndFinalizeNoDPEPModular`: runtime context object and modular prepare/finalize pipeline. / `MoELoRAContext`, `MoEPrepareAndFinalizeNoDPEPModular`：运行时上下文对象与模块化 prepare/finalize 流水线。
- `divide`, `envs`, `_get_lora_device`: tensor-size partitioning, tuned-config toggles, and device placement. / `divide`, `envs`, `_get_lora_device`：张量尺寸切分、调优配置开关与设备选择。
- `maybe_get_oot_by_class`: compatibility with out-of-tree subclasses when selecting replaceable layers. / `maybe_get_oot_by_class`：在选择可替换层时兼容树外子类。
