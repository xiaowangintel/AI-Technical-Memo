# lora_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/ops/xpu_ops/lora_ops.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provide Python wrappers around custom XPU LoRA operators for shrink and expand paths. / 为 shrink 与 expand 路径提供围绕 XPU 自定义 LoRA 算子的 Python 包装函数。

## Line-by-Line Analysis / 逐行分析
### Thin shrink wrapper
```python
def bgmv_shrink(
    inputs: torch.Tensor,
    lora_a_weights: torch.Tensor,
    output_tensor: torch.Tensor,
    lora_indices_tensor: torch.Tensor,
    scaling: float = 1.0,
) -> None:
    torch.ops._xpu_C.bgmv_shrink(
        output_tensor, inputs, lora_a_weights, lora_indices_tensor, scaling
    )
```
**EN:** `bgmv_shrink` is a minimal bridge that forwards tensors and the LoRA scaling factor to the compiled XPU custom op.
**CN:** `bgmv_shrink` 是一个极简桥接函数，把张量和 LoRA 缩放因子直接转发给编译后的 XPU 自定义算子。

### Expand wrapper with dimension mismatch handling
```python
def bgmv_expand(...):
    weight_out_dim = lora_b_weights.size(-2)
    output_dim = output_tensor.size(1)
    if weight_out_dim == output_dim:
        torch.ops._xpu_C.bgmv_expand(...)
    elif weight_out_dim < output_dim:
        torch.ops._xpu_C.bgmv_expand_slice(..., 0, weight_out_dim, add_inputs)
    else:
        lora_b_weights = lora_b_weights[..., :output_dim, :].contiguous()
        torch.ops._xpu_C.bgmv_expand_slice(..., 0, output_dim, add_inputs)
```
**EN:** `bgmv_expand` chooses the correct backend op depending on output shape compatibility. It supports exact matches, partial writes into a larger output buffer, and truncation when weights are wider than the destination.
**CN:** `bgmv_expand` 会根据输出形状兼容性选择合适的后端算子：既支持完全匹配，也支持在更大的输出缓冲区中做局部写入，还支持在权重输出维更大时先截断再写入。

### Explicit sliced expand API
```python
def bgmv_expand_slice(..., slice_offset: int, slice_size: int, add_inputs: bool = True) -> None:
    assert slice_size == lora_b_weights.size(-2)
    assert slice_offset + slice_size <= output_tensor.size(1)
    torch.ops._xpu_C.bgmv_expand_slice(
        output_tensor,
        inputs,
        lora_b_weights,
        lora_indices_tensor,
        slice_offset,
        slice_size,
        add_inputs,
    )
```
**EN:** `bgmv_expand_slice` is the explicit partial-write interface. Its assertions guarantee that the selected slice fits the destination tensor before calling the XPU kernel.
**CN:** `bgmv_expand_slice` 是显式的局部写接口。调用 XPU 内核前，它通过断言确保所选切片与目标张量边界一致。

## Key Concepts / 关键概念
- EN: These wrappers are intentionally thin because the heavy lifting lives in the `_xpu_C` extension. / CN: 这些包装函数刻意保持轻量，因为主要计算逻辑位于 `_xpu_C` 扩展中。
- EN: `bgmv_expand` contains compatibility logic to mirror common PyTorch-side output-shape handling. / CN: `bgmv_expand` 包含兼容性逻辑，以复现 PyTorch 侧常见的输出形状处理方式。

## Dependencies / 依赖关系
- EN: Depends on `torch` and the custom XPU operator namespace `torch.ops._xpu_C`; initializes a logger but does not use it inside the current functions. / CN: 依赖 `torch` 和自定义 XPU 算子命名空间 `torch.ops._xpu_C`；虽然初始化了 logger，但当前函数中并未实际使用。
