# row_parallel_linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/layers/row_parallel_linear.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implement LoRA wrappers for row-parallel linear layers, including a sharded LoRA variant. / 为行并行线性层实现 LoRA 包装器，并包含分片 LoRA 变体。

## Line-by-Line Analysis / 逐行分析
### Row-parallel wrapper setup / 行并行包装器初始化
```python
class RowParallelLinearWithLoRA(BaseLinearLayerWithLoRA):
    def __init__(self, base_layer: RowParallelLinear) -> None:
        super().__init__(base_layer)
```
**EN:** This wrapper adapts the shared linear LoRA machinery to row-parallel layers. Initialization keeps the base layer metadata but reinterprets sizes according to row-wise tensor partitioning.
**CN:** 该包装器把共享线性 LoRA 机制适配到行并行层上。初始化时保留基础层元数据，但会按照按行切分的张量并行方式重新解释尺寸。

### Slice LoRA A by local shard / 按本地分片切分 LoRA A
```python
    def slice_lora_a(self, lora_a: torch.Tensor) -> torch.Tensor:
        shard_size = self.input_size
        start_idx = self.tp_rank * shard_size
```
**EN:** `slice_lora_a` keeps only the input rows owned by the current tensor-parallel rank. This aligns the adapter’s input projection with the row-parallel partition of the base weight.
**CN:** `slice_lora_a` 只保留当前张量并行 rank 所拥有的输入行。这使适配器的输入投影与基础权重的行并行分片保持一致。

### Standard forward path / 标准前向路径
```python
    def forward(
        self, input_: torch.Tensor
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor | None]:
```
**EN:** The normal forward path may split inputs, run the base quantized linear computation, apply the LoRA contribution, and then all-reduce outputs when required.
**CN:** 标准前向路径可能会先切分输入，执行基础量化线性计算，再叠加 LoRA 输出，并在需要时对结果做 all-reduce。

### Sharded LoRA specialization / 分片 LoRA 特化
```python
class RowParallelLinearWithShardedLoRA(RowParallelLinearWithLoRA):
    """
    Differs from RowParallelLinearWithLoRA by slicing the
```
**EN:** The sharded variant changes how LoRA B and the output dimension are partitioned so that adapter work scales better with row-parallel execution.
**CN:** 这个分片变体改变了 LoRA B 以及输出维度的切分方式，使适配器计算更适合行并行执行。

### Sharded apply path / 分片 apply 路径
```python
    def apply(self, x: torch.Tensor, bias: torch.Tensor | None = None) -> torch.Tensor:
        output = self.base_layer.quant_method.apply(self.base_layer, x, bias)
```
**EN:** The overridden `apply` computes the base output first and then injects the sharded LoRA contribution in a way that matches the partial output layout on each rank.
**CN:** 重写后的 `apply` 先计算基础输出，再以匹配各 rank 局部输出布局的方式注入分片 LoRA 增量。

## Key Concepts / 关键概念
- Row-parallel tensor partitioning / 行并行张量切分
- Rank-local slicing of adapter matrices / 适配器矩阵的 rank 本地切片
- Optional sharded LoRA path for better scaling / 用于更好扩展性的分片 LoRA 路径
- Collective communication after local computation / 局部计算后的集体通信

## Dependencies / 依赖关系
- `torch`
- `transformers.PretrainedConfig`
- `vllm.config.lora.LoRAConfig`
- `vllm.distributed.tensor_model_parallel_all_reduce`
- `vllm.model_executor.layers.linear.RowParallelLinear`
- `vllm.lora.layers.base_linear.BaseLinearLayerWithLoRA`
