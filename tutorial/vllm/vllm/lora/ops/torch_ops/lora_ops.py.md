# lora_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/ops/torch_ops/lora_ops.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provide pure-PyTorch LoRA expand/shrink helpers for batched and sequence-grouped execution. / 提供纯 PyTorch 的 LoRA expand/shrink 辅助函数，支持批量与按序列分组执行。

## Line-by-Line Analysis / 逐行分析
### Sequence-grouped wrappers / 序列分组包装器
```python
def sgmv_expand(...):
    exploded_indices = torch.repeat_interleave(lora_indices_tensor, seq_len_tensor)
    bgmv_expand(inputs, lora_b_weights, output_tensor, exploded_indices, add_inputs)


def sgmv_shrink(...):
    exploded_indices = torch.repeat_interleave(lora_indices_tensor, seq_len_tensor)
    bgmv_shrink(inputs, lora_a_weights, output_tensor, exploded_indices, scaling)
```
**EN:** The SGMV entry points convert per-sequence adapter IDs into per-token IDs with `repeat_interleave`, then delegate to the simpler batched kernels.
**CN:** SGMV 入口先用 `repeat_interleave` 将“每个序列一个 LoRA ID”展开成“每个 token 一个 LoRA ID”，再委托给更简单的批处理内核。

### Expand path / Expand 路径
```python
def bgmv_expand(...):
    selected_loras = lora_b_weights[lora_indices_tensor].to(dtype=output_tensor.dtype)
    if len(selected_loras.shape) == 4:
        selected_loras = selected_loras.squeeze(dim=1)
    inputs = inputs.to(dtype=output_tensor.dtype)
    outputs = torch.einsum("bi, boi -> bo", inputs, selected_loras)
```
**EN:** `bgmv_expand` gathers the LoRA-B matrix for each token, normalizes shape, casts to the output dtype, and performs a batched matrix multiply via `einsum`.
**CN:** `bgmv_expand` 为每个 token 选出对应的 LoRA-B 矩阵，整理张量形状并转换到输出 dtype，然后通过 `einsum` 完成批量矩阵乘。

### Padding-aware writeback / 兼容填充的写回逻辑
```python
limit = output_tensor.shape[0]
if outputs.shape[0] == 1 and output_tensor.shape[0] != 1:
    limit = 1

common_len = min(outputs.shape[1], output_tensor.shape[1])
if add_inputs:
    output_tensor[:, :common_len] += outputs[:limit, :common_len]
else:
    output_tensor[:, :common_len] = outputs[:limit, :common_len]
```
**EN:** The function handles shape mismatches caused by padding. It clips to the common output width and can either accumulate into the destination or overwrite it.
**CN:** 该逻辑处理填充导致的形状不一致问题：只写入公共宽度范围，并根据 `add_inputs` 选择累加到目标张量还是直接覆盖。

### Shrink path / Shrink 路径
```python
def bgmv_shrink(...):
    selected_loras = lora_b_weights[lora_indices_tensor].to(dtype=output_tensor.dtype)
    if len(selected_loras.shape) == 4:
        selected_loras = selected_loras.squeeze(dim=1)
    inputs = inputs.to(dtype=output_tensor.dtype)
    outputs = torch.einsum("bi, boi -> bo", inputs, selected_loras)
    output_tensor[:, : outputs.shape[1]] = scaling * outputs[:]
```
**EN:** The shrink helper uses the same gather-and-multiply pattern, then writes the projected low-rank activations into `output_tensor` with an optional scaling factor.
**CN:** shrink 辅助函数复用相同的“按索引选权重 + 矩阵乘”模式，然后把低秩投影结果按 `scaling` 写入 `output_tensor`。

### Sliced expand / 分片 expand
```python
def bgmv_expand_slice(...):
    selected_loras = lora_b_weights[lora_indices_tensor].to(dtype=output_tensor.dtype)
    inputs = inputs.to(dtype=output_tensor.dtype)
    if len(selected_loras.shape) == 4:
        selected_loras = selected_loras.squeeze(dim=1)
    outputs = torch.einsum("bi, boi -> bo", inputs, selected_loras)
```
**EN:** `sgmv_expand_slice` and `bgmv_expand_slice` support writing a LoRA result into only one output interval, controlled by `slice_offset` and `slice_size`.
**CN:** `sgmv_expand_slice` 与 `bgmv_expand_slice` 允许只把 LoRA 结果写入输出张量的一个区间，该区间由 `slice_offset` 和 `slice_size` 指定。

## Key Concepts / 关键概念
- **EN:** `sgmv_*` converts sequence-level routing into token-level routing.
- **CN:** `sgmv_*` 将序列级路由转换为 token 级路由。
- **EN:** `bgmv_*` performs the actual low-rank projection with `torch.einsum`.
- **CN:** `bgmv_*` 通过 `torch.einsum` 执行实际的低秩投影。
- **EN:** Expand writes LoRA-B results back to the model width; shrink writes LoRA-A results to rank space.
- **CN:** expand 将 LoRA-B 结果写回模型宽度；shrink 将 LoRA-A 结果写入低秩空间。

## Dependencies / 依赖关系
- **EN:** Uses `torch` for indexing, dtype conversion, and batched Einstein summation.
- **CN:** 依赖 `torch` 完成索引、dtype 转换和批量爱因斯坦求和。
- **EN:** Exported by `torch_ops/__init__.py` as the public torch backend.
- **CN:** 由 `torch_ops/__init__.py` 重新导出，作为 torch 后端公开接口。
