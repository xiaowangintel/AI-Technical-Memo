# column_parallel_linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/layers/column_parallel_linear.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements LoRA wrappers for tensor-parallel column-style linear layers, including merged, QKV, and sharded variants. / 为张量并行的列并行线性层实现 LoRA 包装器，包括 merged、QKV 和分片变体。

## Line-by-Line Analysis / 逐行分析
### Helper for shared apply path
```python
def _mcp_apply(x, bias, layer: "ColumnParallelLinearWithLoRA"):
    output = layer.base_layer.quant_method.apply(layer.base_layer, x, bias)
    x = x.view(-1, x.shape[-1])
    output, out_orig_shape = output.view(-1, output.shape[-1]), output.shape
    buffers = torch.empty_strided(...)
    buffers.zero_()
    shrunk_buffers = layer.punica_wrapper.add_shrink(buffers, x, layer.lora_a_stacked, 1.0)
    buffers = tensor_model_parallel_all_gather(buffers)
    lora_output = layer.punica_wrapper.add_expand(
        output, buffers, layer.lora_b_stacked, layer.output_slices,
        offset_start=0, add_input=True,
    )
    return output.view(*out_orig_shape)
```
**EN:** This helper centralizes the LoRA execution path for column-parallel layers. It first runs the base quantized linear op, then computes the LoRA A projection (`add_shrink`), gathers rank-local intermediate buffers across tensor-parallel workers, and finally applies LoRA B (`add_expand`) into the output tensor. The explicit strided buffer avoids layout issues under `torch.compile`.
**CN:** 这个辅助函数统一了列并行层的 LoRA 执行路径。它先执行基础量化线性算子，再通过 `add_shrink` 计算 LoRA A 投影，随后在张量并行进程间收集中间缓冲区，最后用 `add_expand` 将 LoRA B 增量写回输出。显式的 stride 缓冲区用于避免 `torch.compile` 下的布局问题。

### Base wrapper for ColumnParallelLinear
```python
class ColumnParallelLinearWithLoRA(BaseLinearLayerWithLoRA):
    def __init__(self, base_layer: ColumnParallelLinear) -> None:
        self.is_merged_col_linear = isinstance(base_layer, MergedColumnParallelLinear)
        self.output_size = self.base_layer.output_size_per_partition
        self.n_slices = 1

    def slice_lora_b(self, lora_b: torch.Tensor) -> torch.Tensor:
        if self.is_merged_col_linear:
            ...
        else:
            shard_size = self.output_size
            start_idx = self.tp_rank * shard_size
            end_idx = (self.tp_rank + 1) * shard_size
            lora_b = lora_b[start_idx:end_idx, :]
        return lora_b
```
**EN:** The base wrapper handles the common case of a single LoRA attached to a column-parallel linear layer. Its main job is partition-aware slicing of `lora_b`: plain column-parallel layers shard a contiguous output range, while merged layers pack two logical halves and must slice each half separately.
**CN:** 这个基础包装器处理“单个 LoRA + 列并行线性层”的常见情况。其核心职责是按并行分片切分 `lora_b`：普通列并行层直接切连续输出区间，而 merged 层把两个逻辑子层打包在一起，因此需要分别切分两个半区后再拼接。

### Forward path and replacement check
```python
def forward(self, input_: torch.Tensor):
    bias = self.base_layer.bias if not self.base_layer.skip_bias_add else None
    output_parallel = self.apply(input_, bias)
    if self.base_layer.gather_output and self.tp_size > 1:
        output = tensor_model_parallel_all_gather(output_parallel)
    else:
        output = output_parallel
    ...

@classmethod
@_not_fully_sharded_can_replace
def can_replace_layer(cls, source_layer, lora_config, packed_modules_list, model_config=None):
    ...
```
**EN:** `forward` mirrors the original linear layer API: it optionally adds bias, applies LoRA-enhanced computation, and all-gathers the output when the underlying layer expects a gathered tensor. `can_replace_layer` is the selection gate used by the LoRA system to decide whether this wrapper matches a source module.
**CN:** `forward` 保持与原始线性层一致的接口：按需处理 bias，执行带 LoRA 的计算，并在底层层要求聚合输出时进行 all-gather。`can_replace_layer` 是 LoRA 系统选择包装器时的匹配入口。

### Merged column-parallel layers
```python
class MergedColumnParallelLinearWithLoRA(ColumnParallelLinearWithLoRA):
    def __init__(self, base_layer):
        self.output_sizes = self.base_layer.output_sizes
        self.output_slices = tuple(divide(output_size, self.tp_size) for output_size in self.output_sizes)
        self.n_slices = len(self.output_slices)

    def create_lora_weights(self, max_loras, lora_config, model_config=None):
        self.lora_a_stacked = tuple(torch.zeros(...) for _ in range(self.n_slices))
        self.lora_b_stacked = tuple(torch.zeros(...) for output_size in self.output_slices)
```
**EN:** This subclass supports packed layers such as `gate_up_proj`, where one module contains multiple logical projections. It tracks per-slice output widths and allocates separate stacked LoRA A/B tensors for each slice so each logical projection can receive its own adapter.
**CN:** 这个子类用于 `gate_up_proj` 一类打包层，即一个模块内部包含多个逻辑投影。它记录每个 slice 的输出宽度，并为每个 slice 单独分配堆叠的 LoRA A/B 张量，从而让每个逻辑投影都能挂载独立适配器。

### Expanding packed adapter groups and loading weights
```python
def expand_packed_lora(self, lora_a, lora_b):
    for a_i, b_i in zip(lora_a, lora_b):
        ...
        for j in range(covered):
            size = self.output_sizes[start_idx + j]
            expanded_b.append(b_i[start : start + size, :])
            expanded_a.append(a_i)

def set_lora(self, index, lora_a, lora_b):
    self.reset_lora(index)
    if isinstance(lora_b, list) and len(lora_b) != self.n_slices:
        lora_a, lora_b = self.expand_packed_lora(lora_a, lora_b)
    if self.tp_size > 1:
        lora_a = self.slice_lora_a(lora_a)
        lora_b = self.slice_lora_b(lora_b)
    ... copy_ into self.lora_a_stacked / self.lora_b_stacked ...
```
**EN:** Checkpoints do not always store one LoRA tensor per runtime slice. `expand_packed_lora` detects adapters that cover multiple packed outputs and splits `lora_b` according to `output_sizes`, while reusing the same `lora_a`. `set_lora` then shards and copies the resulting tensors into the runtime buffers.
**CN:** 检查点中的 LoRA 张量不一定与运行时 slice 一一对应。`expand_packed_lora` 会识别覆盖多个打包输出的适配器，按 `output_sizes` 拆分 `lora_b`，同时复用同一个 `lora_a`。随后 `set_lora` 再按并行方式切分并拷贝到运行时缓冲区中。

### QKV-specific slicing logic
```python
class QKVParallelLinearWithLoRA(ColumnParallelLinearWithLoRA):
    def slice_lora_b(self, lora_b: torch.Tensor) -> torch.Tensor:
        lora_b_q = lora_b[...]
        lora_b_k = lora_b[...]
        lora_b_v = lora_b[...]
        return torch.cat([lora_b_q, lora_b_k, lora_b_v], dim=0)

class MergedQKVParallelLinearWithLoRA(MergedColumnParallelLinearWithLoRA):
    self.output_slices = (self.q_proj_shard_size, self.kv_proj_shard_size, self.kv_proj_shard_size)
    self.output_ids = (self.q_shard_id, self.kv_shard_id, self.kv_shard_id)
```
**EN:** QKV layers need asymmetric handling because Q, K, and V may have different shard sizes, especially with grouped-query attention. These classes compute separate shard IDs for Q versus KV and slice/reassemble LoRA weights accordingly.
**CN:** QKV 层需要特殊处理，因为 Q、K、V 的分片大小可能不同，尤其在 grouped-query attention 下更明显。这些类分别计算 Q 与 KV 的 shard ID，并据此切分和重组 LoRA 权重。

### Fully sharded LoRA variants
```python
class ColumnParallelLinearWithShardedLoRA(ColumnParallelLinearWithLoRA):
    def slice_lora_a(self, lora_a: torch.Tensor) -> torch.Tensor:
        shard_size = self.lora_a_stacked[0].shape[2]
        start_idx = self.tp_rank * shard_size
        return lora_a[start_idx : start_idx + shard_size, :]

    def apply(self, x, bias=None):
        return _mcp_apply(x, bias, self)
```
**EN:** The sharded subclasses implement the S-LoRA strategy: in addition to sharding `lora_b`, they also shard `lora_a` along the rank dimension. The concrete subclasses reuse the same execution helper but override slicing and replacement policy for fully sharded configurations.
**CN:** 这些分片子类实现了 S-LoRA 策略：除了切分 `lora_b` 外，还会沿 rank 维切分 `lora_a`。具体子类复用相同的执行辅助函数，但会覆盖切分逻辑与 fully sharded 场景下的替换策略。

### Variable-slice merged layers
```python
class MergedColumnParallelLinearVariableSliceWithLoRA(MergedColumnParallelLinearWithLoRA):
    @classmethod
    def can_replace_layer(...):
        if len(packed_modules_list) >= 3:
            return True
        ...

    def set_lora(self, index, lora_a, lora_b):
        if isinstance(lora_a, torch.Tensor):
            lora_a = [lora_a] * self.n_slices
        if isinstance(lora_b, torch.Tensor):
            ... split by output_sizes ...
        super().set_lora(index, lora_a, lora_b)
```
**EN:** This final class extends support to merged column-parallel layers with three or more slices. It also handles checkpoints that save a single combined tensor by duplicating `lora_a` and splitting `lora_b` according to the layer’s `output_sizes` before delegating to the parent loader.
**CN:** 最后这个类把支持范围扩展到三段及以上的 merged 列并行层。它还兼容“检查点只保存单个合并张量”的情况：先复制 `lora_a`，再按层的 `output_sizes` 拆分 `lora_b`，最后交给父类完成装载。

## Key Concepts / 关键概念
- **Tensor-parallel LoRA slicing**: `lora_b` is sharded by output partition; fully sharded modes also shard `lora_a` by rank. / **张量并行 LoRA 切分**：`lora_b` 按输出分片切分；fully sharded 模式还会沿 rank 切分 `lora_a`。
- **Packed linear layers**: merged modules may represent multiple logical projections inside one weight tensor. / **打包线性层**：merged 模块可能在一个权重张量里包含多个逻辑投影。
- **QKV asymmetry**: query shards and key/value shards may use different sizes and replica rules. / **QKV 非对称性**：Q 分片与 K/V 分片的大小和复制规则可能不同。
- **Runtime buffer layout**: the helper carefully shapes intermediate buffers so custom kernels and `torch.compile` both work correctly. / **运行时缓冲区布局**：辅助函数精心构造中间缓冲区，以同时兼容自定义内核和 `torch.compile`。
- **Wrapper selection**: `can_replace_layer` methods encode which runtime layer shape maps to which LoRA wrapper. / **包装器选择**：`can_replace_layer` 方法定义了运行时层结构如何映射到对应 LoRA 包装器。

## Dependencies / 依赖关系
- `torch`, `torch.nn`: tensor storage, slicing, and async `copy_`. / `torch`, `torch.nn`：张量存储、切分和异步 `copy_`。
- `PretrainedConfig`, `LoRAConfig`: model metadata and LoRA sizing options. / `PretrainedConfig`, `LoRAConfig`：模型元数据与 LoRA 尺寸配置。
- `tensor_model_parallel_all_gather`, `divide`: tensor-parallel communication and size partitioning. / `tensor_model_parallel_all_gather`, `divide`：张量并行通信与尺寸分配。
- `ColumnParallelLinear`, `MergedColumnParallelLinear`, `QKVParallelLinear`: wrapped base layer types. / `ColumnParallelLinear`, `MergedColumnParallelLinear`, `QKVParallelLinear`：被包装的底层层类型。
- `BaseLinearLayerWithLoRA`, `_fully_sharded_can_replace`, `_not_fully_sharded_can_replace`: shared LoRA-layer machinery and replacement policies. / `BaseLinearLayerWithLoRA`, `_fully_sharded_can_replace`, `_not_fully_sharded_can_replace`：共享 LoRA 层机制与替换策略。
- `current_platform`, `maybe_get_oot_by_class`: platform-specific behavior and compatibility with out-of-tree subclasses. / `current_platform`, `maybe_get_oot_by_class`：平台相关行为与对树外子类的兼容支持。
